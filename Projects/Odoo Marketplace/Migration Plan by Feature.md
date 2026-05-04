---
aliases:
  - Odoo Marketplace
tags:
  - dev/erp
  - dev/backend
date: 2026-05-04
---
**Related:** [[Odoo Marketplace]]

---

# odoo_marketplace — Odoo 17 → 19 Migration Plan (Feature-First)

## Why Feature-First, Not Layer-First

A layer-first migration ("all models, then all views, then all controllers") is tempting because it mirrors the module directory structure. It fails in practice for one reason: you cannot test a model without its security rules, you cannot test security rules without its groups, and you cannot test groups without knowing whether the partner is a seller. Every feature in this module cross-cuts at least 4 layers (model, security, views, controller). Migrating by layer means you spend the first 60% of the project with zero working functionality and zero feedback from tests.

A feature-first migration delivers one vertical slice at a time. Each slice is installable, testable, and provably working before you touch the next one. If you get blocked on Phase 7, you still have 6 working features.

The recommended order is also a dependency topological sort: each phase only depends on phases already completed.

## The Dependency Map (from the knowledge graph)

The graph has one critical fact: ResPartner has 43 edges — more than ProductTemplate (24) and SaleOrderLine (19) combined. It is the absolute god node. Every other feature either reads partner.seller, writes partner.state, or navigates to partner.seller_shop_id. If ResPartner's seller extensions are broken, the entire module breaks silently (wrong domain filters return wrong records; no exceptions, just wrong data).

The community structure confirms the feature groupings:

ResPartner (43 edges)
└── drives → ProductTemplate (24) → SaleOrderLine (19) → SellerPayment (16) → StockPicking
└── drives → SellerShop (15) → SellerReview
└── drives → AuthSignupHome (13) → web routes

## Phase 0 — Pre-Migration Infrastructure

Files: **__init__.py**, **__manifest__.py**, **models/mp_tools.py**

### What to do

1. Fix the version gate (**__init__.py:27**):
```diff
- # Current — blocks Odoo 19 installation
- if not 16.0 < float(server_serie) <= 17.0:
+ # Target — allows Odoo 19 installation
+ if not 18.0 < float(server_serie) <= 19.0:
```

This single line blocks the entire module from loading on Odoo 19. Nothing else matters until this is fixed.

2. Audit **__manifest__.py** dependencies: website_sale_stock, stock_account, delivery, sale_management — verify each still exists in Odoo 19. website_sale_stock was merged into website_sale in Odoo 18.

3. **mp_tools.py**: Review any internal Odoo utility it imports. Utilities tend to move between private and public APIs across major versions.


### Why first

Nothing installs without this. Zero feedback from the Odoo server until the version gate passes.


## Phase 1 — Core Identity: ResPartner + ResUsers + Security Skeleton

Files: models/res_partner.py, models/res_users.py, security/marketplace_security.xml (groups only, no record rules yet), security/ir.model.access.csv (minimum rows for res.partner and res.users)

### What to do

ResPartner is the god node (43 edges). Its seller extensions are read by every other model:

- partner.seller — boolean gate for every domain filter
- partner.state — new/pending/approved/denied drives group membership
- partner.commission — used in SOL and AccountMove commission calculation
- partner.warehouse_id / partner.location_id — used in StockPicking splitting
- partner.seller_shop_id — used in frontend routing


The critical breaking change here is \_read_group_fill_results at res_partner.py:181. This method was removed from Odoo's ORM in Odoo 18. It was a private kanban state-filling hook. The replacement is overriding \_read_group with the new signature:

```diff
- # Old (Odoo 17) — REMOVE entirely
- def \_read_group_fill_results(self, domain, groupby, remaining_groupbys, ...):
    return super().\_read_group_fill_results(...)
+ # New (Odoo 18+) — only needed if you want custom group ordering
+ @api.model
+ def \_read_group_stage_ids(self, stages, domain, order):
+ ...
```

This affects 8 models across the module. Fix res_partner.py here as the template; repeat the same fix in each subsequent phase.

ResUsers group management methods (config_wise_groups_status, update_groups_for_mp_user) depend on group XML IDs that must exist. Migrate the security group hierarchy first:

```mermaid
flowchart TD
    Draft[marketplace_draft_seller_group] --> Seller[marketplace_seller_group]
    Seller --> Officer[marketplace_officer_group]
    Officer --> Manager[marketplace_manager_group]
    Manager --> Shop[group_marketplace_seller_shop (hidden)]
```

Security access rules for res.partner: Add the minimum rows to ir.model.access.csv for Pending Seller, Seller, Officer, and Manager on res.partner. Without these, any  
 test that touches partner records will throw AccessError regardless of correctness.

Do not migrate record rules yet — those require the groups to exist and the model fields to be present.

Why first

ResPartner has 43 edges. Every single feature in every subsequent phase reads from it. If partner.seller is broken, kanban views return wrong records, domain filters  
 silently exclude the right partners, and email templates address the wrong recipients. This is the foundation everything else sits on.

---

Phase 2 — Marketplace Configuration

Files: models/res_config.py, models/website.py, views/res_config_view.xml, views/website_config_view.xml, data/mp_config_setting_data.xml

What to do

Migrate ResConfigSettings with all mp\_\* fields. This model (340 lines) controls:

- Auto-approval toggles for sellers and products
- Global commission percentage
- Payment limits and journal
- Which website features are enabled (seller lists, reviews, shop, become-seller button)
- Email template references  


The website.py extension adds mp\_\* fields to website that the frontend templates read.

Migrate the XML configuration views. These are 58KB of form view — largely stable across Odoo versions, but check for any attrs= patterns and replace with direct  
 invisible= attributes.

Load mp_config_setting_data.xml data — this seeds default email template IDs into the config. Without it, notification emails crash when templates are False.

Why second (after identity)

Every feature has a config toggle. Auto-approve product? Config field. Commission rate? Config field. If config is not migrated, you spend the next 10 phases hunting  
 bugs that are actually "feature disabled by default with no visible error".

---

Phase 3 — Pending Seller Registration

Files: models/res_partner.py (registration methods), models/res_users.py (signup flow), controllers/main.py (routes: seller_signup_form, become_seller,
submit_as_seller), views/website_account_template.xml, edi/ (seller creation email templates), wizard/seller_registration_wizard.py + view, security/ (record rules for
marketplace_draft_seller_group)

What to do

This is the entry point of the entire user journey. The flow is:

1. Anonymous user visits /seller/signup → AuthSignupHome.seller_signup_form()
2. User fills form → partner created with seller=True, state='new'
3. User clicks "Become Seller" → become_seller() renders the terms page
4. User submits → submit_as_seller() calls set_to_pending() on the partner
5. Partner assigned to marketplace_draft_seller_group
6. Email sent to admin (seller_creation_mail_to_admin)
7. Email sent to seller (seller_creation_mail_to_seller)  


Breaking change — website controller: The AuthSignupHome extends Odoo's login controller. In Odoo 18/19, the auth signup flow was refactored (/web/signup endpoint  
 changes). The web_login() method override at controllers/main.py:117 must be re-verified against the new base controller signature.

Record rules for Pending Seller: Add the marketplace_draft_seller_group record rules now. Pending sellers can only see their own partner record. These rules use  
 user.partner_id.id comparisons which are stable across versions.

Email templates: mail.template in Odoo 18+ changed email_from computation. Templates that relied on ${object.company_id.email} may need updating.

Why third

This is the first end-to-end user journey. It only depends on: ResPartner seller fields (Phase 1), security groups (Phase 1), and configuration (Phase 2). Getting this
working first gives you a testable registration flow and proves the auth controller integration is working.

---

Phase 4 — Seller Approval Flow

Files: models/res_partner.py (set_to_pending, approve, deny, change_seller_group_and_send_mail, change_seller_group), models/res_users.py (update_groups_for_mp_user),
wizard/seller_status_reason.py + view, views/seller_view.xml, edi/ (seller status change templates), security/ (record rules for marketplace_seller_group)

What to do

The state machine: new → pending → approved/denied → (denied can re-apply)

When a seller is approved:

- ResPartner.approve() calls change_seller_group_and_send_mail()
- change_seller_group() calls ResUsers.update_groups_for_mp_user()
- The user is moved from marketplace_draft_seller_group to marketplace_seller_group
- Email sent via seller_status_change_mail_to_seller  


The seller_status_reason.py wizard captures the reason when denying a seller — test this path explicitly.

Add marketplace_seller_group record rules now: Sellers can see their own SOL, their own products, etc. Add these rules at the security layer for the seller group even  
 though the models they govern (products, orders, payments) aren't migrated yet — Odoo loads rules lazily and rules for non-existent fields don't crash.

Why fourth

Seller approval is the prerequisite gate for all remaining features. Products can only be created by approved sellers. Shops require an approved seller. Orders are only
visible to sellers who are in marketplace_seller_group. If this state machine is broken, you cannot test a single subsequent feature with real seller-scoped data.

---

Phase 5 — Dynamic Menu & Action Control

Files: models/ir_ui_menu.py, models/ir_action.py, models/ir_attachment.py, static/src/js/ (backend UI restriction scripts), views/mp_menu_view.xml

What to do

These three model extensions implement the "seller sees only seller menus" pattern:

- IrUiMenu.\_visible_menu_ids() filters backend menus for seller group users
- IrActWindow restricts actions available to sellers
- IrAttachment restricts which attachments sellers can access

The url_handler.js, clickable_off.js, right_click_prevent.js backend assets disable right-click and certain URL patterns for seller users.

Migrate the Seller Dashboard menu tree at this point — all top-level menus with group restrictions.

Note: In Odoo 18+, the backend menu system uses a different rendering path. Test with an actual seller user after migration to confirm the menu restriction is still  
 applying.

Why fifth

This phase has minimal model dependencies (only user groups from Phase 1). But sellers who just got approved (Phase 4) need to experience the correct restricted UI  
 before you add feature-specific menus in subsequent phases. Also, ir_attachment.py restriction is a security control — migrate security controls early.

---

Phase 6 — Seller Shop

Files: models/seller_shop.py (SellerShop, SellerShopStyle), models/seller_social_media.py, models/res_partner.py (social media links methods),
views/seller_shop_view.xml, views/website_seller_shop_template.xml, controllers/main.py (MarketplaceSellerShop class routes), data/seller_shop_style_data.xml,  
 data/social_media_data.xml, security/ (SellerShop record rules + access CSV rows)

What to do

SellerShop has 15 edges in the graph and acts as the seller's public-facing container. The create() override must be validated — it likely auto-creates a shop when a  
 seller is approved.

SellerShop.website_published and website_sequence are used for next/previous navigation. In Odoo 18+, website_published was renamed to is_published (from  
 website.published.mixin). This is a silent breaking change — the field still exists as an alias in many Odoo 18 versions but the mixin inheritance chain changed. Verify
this explicitly:

# Old

\_inherit = ['website.published.mixin']

# Check Odoo 19's website.published.mixin still provides website_published

# or switch all references to is_published

The controller routes /seller/shop/<handler> and /seller/shops/list/ must be tested. The handler slug generation uses partner.url_handler which is a res.partner field —
confirm this survives the ResPartner migration.

Why sixth

SellerShop is the profile container. Products (Phase 7) link to shops. Reviews (Phase 11) display on shop pages. The controller routes for shops are simpler than  
 product pages and have no accounting dependencies, so this is the right point to validate the website routing before adding product complexity.

---

Phase 7 — Product Management

Files: models/marketplace_product.py (ProductTemplate and ProductProduct extensions), wizard/action_wizard.py, wizard/publish.py, wizard/unpublish.py,
wizard/variant_approval_wizard.py, views/mp_product_view.xml (44KB), security/ (product record rules + access CSV rows), data/mp_product_demo_data.xml

What to do

Products are the central marketplace asset (24 edges). The status state machine: draft → pending → approved/rejected → draft.

Critical breaking change — \_read_group_fill_results in marketplace_product.py:65: Remove this override entirely. In Odoo 18+, the kanban column ordering for the status
field should use \_group_by_full or the new \_read_group override depending on your Odoo 19 version.

Critical: toggle_website_published() is overridden three times (ProductTemplate and ProductProduct both override it, plus ResPartner). The guard logic at  
 marketplace_product.py:124 prevents sellers from publishing unapproved products. This business rule must survive the is_published migration — if you rename
website_published to is_published, these guards must be updated simultaneously.

website_sale_stock context key at marketplace_product.py:354:  
 if not self.env.context.get('website_sale_stock_get_quantity'):
This context key may have changed or been removed in the website_sale_stock → website_sale merge. Verify against Odoo 19's website_sale module.

Direct import in sale.py:21 (will be needed in Phase 8):  
 from odoo.addons.website_sale_stock.models.sale_order import SaleOrder as WebsiteSaleStock  
 If website_sale_stock was merged into website_sale in Odoo 18/19, this import crashes at startup. Investigate early here even though it's in sale.py.

The mp_product_view.xml is 44KB and likely contains many attrs= patterns from Odoo 16 patterns. Do a full replacement:

  <!-- Old -->
  <field name="status" attrs="{'invisible': [('marketplace_seller_id', '=', False)]}"/>                                                                                   
  <!-- New (Odoo 17+) -->                                                                                                                                                 
  <field name="status" invisible="not marketplace_seller_id"/>                                                                                                            
   
  Why seventh                                                                                                                                                             
                  
  Products are the first revenue-generating asset. They depend on approved sellers (Phase 4) and optionally shops (Phase 6). Every subsequent feature — orders, stock,    
  payments — requires products to exist. Migrating products before orders means you can populate test data correctly.
                                                                                                                                                                          
  ---             
  Phase 8 — Order Management (Sale Order + Order Lines)
                                                                                                                                                                          
  Files: models/sale.py (SaleOrder and SaleOrderLine extensions), wizard/mark_done.py, wizard/mark_approved.py, wizard/bulk_sol_confirm_wizard.py, views/mp_sol_view.xml,
  security/ (SOL record rules + access CSV rows)                                                                                                                          
                  
  What to do                                                                                                                                                              
                  
  SaleOrderLine has 19 edges. The marketplace_state field state machine (new → pending → approved → shipped → done → cancel) drives both the stock update (StockMove sets 
  it to "shipped") and the payment calculation (only "done" lines count for seller_amount).
                                                                                                                                                                          
  _read_group_fill_results in sale.py:211: Same removal as Phase 7. This is the kanban groupby for marketplace_state.                                                     
   
  SaleOrder.write() warehouse override: When an order is confirmed, SaleOrder overrides the warehouse to use seller.warehouse_id. This depends on res.partner seller      
  fields (Phase 1) — verify the warehouse lookup still works.
                                                                                                                                                                          
  admin_commission and seller_amount compute fields: These depend on res.config.settings global commission (Phase 2) and per-seller partner.commission (Phase 1). Test    
  with both global and per-seller commission configurations.
                                                                                                                                                                          
  Bulk action wizards (mark_done, mark_approved, bulk_confirm): These operate on recordsets of SOL and change marketplace_state. They must be migrated with their XML     
  views at the same time — a wizard without its view breaks the entire action.
                                                                                                                                                                          
  website_sale_stock import: The from odoo.addons.website_sale_stock... import at line 21 of sale.py must be resolved here. If the module was merged, find the equivalent 
  class in website_sale and update the import or remove the inheritance if the method was upstreamed.
                                                                                                                                                                          
  Why eighth

Orders are triggered by product purchases (Phase 7). The SOL marketplace_state machine is the trigger for both stock updates (Phase 9) and payment creation (Phase 10).
You cannot test stock or payments without working order lines.

---

Phase 9 — Inventory & Stock Management

Files: models/stock.py (MarketplaceStock, StockPicking, StockMove), views/mp_stock_view.xml (55KB), security/ (stock record rules + access CSV rows for
marketplace.stock, stock.picking, stock.move)

What to do

This phase has three independent components that interact:

MarketplaceStock — seller inventory requests (draft → requested → approved/rejected). When approved, calls change_product_qty() on stock.quant. In Odoo 18+, stock.quant
inventory adjustment API changed (\_update_available_quantity vs direct write). Verify the quantity update method.

\_read_group_fill_results in stock.py:99 and stock.py:228: Two separate overrides — one on MarketplaceStock, one on StockPicking. Both must be removed and replaced.

StockPicking.button_validate() override at stock.py:280: This is where the seller's picking confirms delivery and triggers \_send_confirmation_email(). The  
 stock.picking.button_validate() method signature changed in Odoo 18 — it became an async-capable action. Verify the super() call chain.

StockMove.write() at stock.py:344: When a stock move is done, it updates sale.order.line.marketplace_state to "shipped". This cross-model write is the tightest coupling
in the module. Test this end-to-end: create an order, confirm it, validate the delivery, verify SOL state changes.

\_read_group_fill_results on StockPicking: The stock picking kanban view groups by seller. This override must be replaced.

The mp_stock_view.xml is 55KB. Same attrs= → invisible= migration as Product views.

Why ninth

Stock is downstream of orders (Phase 8) — it's triggered by SO confirmation. Stock completion (StockMove.write) upstream-updates SOL state, which affects payment  
 calculation. The inventory request flow is independent but uses the same models. Both must be working before you can prove the payment calculation is correct.

---

Phase 10 — Payment Management

Files: models/seller_payment.py, models/seller_payment_method.py, models/account_move.py, wizard/seller_payment_wizard.py, wizard/account_payment_register.py,
views/seller_payment_view.xml, views/account_invoice_view.xml, security/ (seller.payment record rules + access CSV, account.move seller rules),  
 data/seller_payment_method_data.xml

What to do

Payment is the most complex integration in the module. It spans 4 models and has 4 trigger points:

1. Invoice confirmed → AccountMove.\_action_invoice_posted() override → create_seller_payment_new() creates seller.payment in credit mode
2. Seller requests payment → SellerPaymentWizard.do_request() validates amount, creates seller.payment in debit mode
3. Payment confirmed → SellerPayment.do_Confirm() creates vendor bill (account.move)
4. Vendor bill paid → AccountMove.write() detects payment_state == 'paid' → sets seller.payment.state = 'posted'  


\_read_group_fill_results in seller_payment.py:91: Remove and replace.

account.move API changes in Odoo 18/19: This is the highest-risk integration. Odoo 18 changed:

- \_action_invoice_posted() was renamed/refactored
- payment_state field computation was updated
- Vendor bill creation vals dict keys may have changed (specifically move_type, invoice_line_ids)
- The account.payment.register wizard API changed  


Check each account.move.create(vals) call in account_move.py:156 and verify the vals dict matches Odoo 19's account.move field list.

SellerPaymentWizard.validate_payment_request() checks minimum payment gap and payment limit from config (Phase 2) — verify ResConfigSettings field names are consistent.

seller_payment_method_data.xml: 5 payment methods seeded at install. No breaking changes expected, but verify the sequence number format hasn't changed.

Why tenth

Payments depend on everything: approved sellers (Phase 4), delivered orders (Phase 8 + 9), and working accounting. It is also the feature with the most  
 Odoo-version-sensitive code (account.move). Migrating it last among the core transactional features means you can mock with real test data from the preceding phases and
immediately know if commission calculations are correct.

---

Phase 11 — Seller Reviews & Recommendations

Files: models/seller_review.py (SellerReview, ReviewHelp, SellerRecommendation), views/seller_review_view.xml, controllers/main.py (SellerReview class routes),
security/ (review access rules for portal and public groups)

What to do

Reviews are the most self-contained feature (cohesion score 0.08 in the graph, community 6). They only need an approved seller to exist.

\_read_group_fill_results in seller_review.py:138 and seller_review.py:239: Two overrides, one per model. Remove both.

website_published on SellerReview: The review has website_published field with a computed is_published field layered on top (seller_review.py:97-127). This double-field
pattern is fragile. In Odoo 19, consolidate to use only is_published from the mixin if SellerReview inherits website.published.mixin.

Portal access rules: Reviews are readable by base.group_portal and writable by portal users (customers writing reviews). The review() and review_help() controller  
 endpoints use request.env['seller.review'].sudo() — one of the 80 sudo() calls. In Odoo 18+, portal sudo() patterns should be replaced with
request.env['seller.review'].with_user(request.env.ref('base.public_user')) or proper portal record rules.

JSON endpoints: review() and seller_recommend() return JSON. Verify the http.route response format — Odoo 18 changed some JSON response wrappers.

Why eleventh

Reviews only depend on seller existence and don't block any transactional feature. They can be migrated after the revenue-critical features (orders, stock, payments)  
 are stable. They're also useful for visual testing of the seller profile page in Phase 13.

---

Phase 12 — Marketplace Dashboard

Files: models/marketplace_dashboard.py, views/mp_dashboard_view.xml (66KB), data/marketplace_dashboard_demo.xml

What to do

The dashboard is a pure read-only aggregation model. It has 9 edges in the graph — all of them reading data from other models. The kanban view (66KB) displays counts:  
 products, sellers, orders, payments, stock requests.

The marketplace_dashboard.py model computes counts using search_count() calls filtered by domain. These are stable API calls that don't change across versions.

The 66KB view is the largest in the module. It likely contains significant amounts of attrs= patterns and possibly <tree> tags. Do a full find-and-replace pass:  
 grep -c "attrs=" views/mp_dashboard_view.xml
grep -c "<tree" views/mp_dashboard_view.xml

The dashboard's kanban "state" grouping may have used \_read_group_fill_results indirectly through a related model. Verify by loading the kanban view after all previous
phases are installed.

Why twelfth

The dashboard has zero data of its own — it reads from all other models. It can only be fully tested when all previous phases are working. It's the "health check" of  
 the whole module.

---

Phase 13 — Web Frontend: Controllers + Templates

Files: controllers/main.py (remaining routes: MarketplaceSellerProfile, load*mp_all_seller, seller_profile_recently_product, user_avatar, mp_sell, add_header_button,
portal invoice override), all views/website*\*.xml templates, views/snippets/, static/src/ (all CSS/JS frontend assets)

What to do

This is the largest phase by file size but the least risky by business logic. All backend data is already correct from Phases 1–12.

Critical: website_sale dependency: Every product page, cart integration, and inventory check goes through website_sale. In Odoo 18, website_sale was significantly  
 refactored (new checkout flow, product page rebuilding, cart management API changes). The marketplace module's product template website_mp_product_template.xml adds
seller info to product pages — verify the XPath selectors still find the right anchor elements.

website_marketplace_dashboard controller renders the seller's personal dashboard at /my/marketplace. The portal integration (PortalAccount override at  
 controllers/main.py:1086) overrides portal_my_invoices(). In Odoo 18/19, the portal invoice list view changed its template structure — verify XPaths in the template
inheritance.

AuthSignupHome and web_login(): This was partially addressed in Phase 3. Complete the migration here — test the full login-redirect flow for seller users.

80 sudo() calls in controllers: Audit all of them. Group them by purpose:

- Legitimate: public pages reading seller profiles (seller has no session)
- Risky: writes done under sudo (review creation, recommendation votes) — add explicit access checks before the sudo  


Frontend assets: marketplace.js and review_chatter.js use jQuery and older OWL patterns. Odoo 18/19 moved further toward pure OWL. Check if any deprecated JS API calls
were removed.

Snippets: sell_snippets.xml adds website builder snippets. The snippet API changed in Odoo 18 — options, data-selector, and the snippet manager API all evolved. Test in
the website editor.

Why last

Frontend depends on all backend data being correct. Template XPaths reference view IDs from backend views — if those views changed during earlier phases, the XPaths  
 would be wrong. Testing frontend last means you get real data from real flows, not mocked data.

---

Cross-Cutting Breaking Changes Affecting All Phases

These must be applied globally, not per-phase. Track them as a separate checklist alongside feature work:

┌─────────────────────────────┬───────────────────────────────────────────────────────┬──────────────────────────┬──────────────────────────────────────────────────┐  
 │ Issue │ Scope │ Impact │ Fix │  
 ├─────────────────────────────┼───────────────────────────────────────────────────────┼──────────────────────────┼──────────────────────────────────────────────────┤  
 │ \_read_group_fill_results │ res_partner.py, marketplace_product.py, sale.py, │ Crash at runtime — 8 │ Remove overrides; use \_read_group new signature │
│ removed │ seller_payment.py, seller_review.py, stock.py (2×) │ call sites │ if column ordering is needed │
├─────────────────────────────┼───────────────────────────────────────────────────────┼──────────────────────────┼──────────────────────────────────────────────────┤  
 │ Version gate in │ **init**.py:27 │ Blocks all installation │ Change <= 17.0 to <= 19.0 │
│ pre_init_hook │ │ │ │  
 ├─────────────────────────────┼───────────────────────────────────────────────────────┼──────────────────────────┼──────────────────────────────────────────────────┤
│ website_sale_stock module │ sale.py:21, **manifest**.py │ ImportError at startup │ Update import to website_sale equivalents │  
 │ merge │ │ │ │  
 ├─────────────────────────────┼───────────────────────────────────────────────────────┼──────────────────────────┼──────────────────────────────────────────────────┤
│ website_published → │ res_partner.py, marketplace_product.py, │ Silent wrong behavior │ Migrate to is_published from │  
 │ is_published │ seller_shop.py, seller_review.py │ │ website.published.mixin │  
 ├─────────────────────────────┼───────────────────────────────────────────────────────┼──────────────────────────┼──────────────────────────────────────────────────┤
│ attrs= deprecated in XML │ 34 occurrences in views │ Works in 17, warnings in │ Replace with direct │  
 │ │ │ 18, may break in 19 │ invisible=/required=/readonly= │  
 ├─────────────────────────────┼───────────────────────────────────────────────────────┼──────────────────────────┼──────────────────────────────────────────────────┤
│ account.move API changes │ account_move.py │ Payment flow may break │ Verify all create() vals dicts and │  
 │ │ │ │ \_action_invoice_posted() hook │  
 ├─────────────────────────────┼───────────────────────────────────────────────────────┼──────────────────────────┼──────────────────────────────────────────────────┤
│ │ │ Security risk + │ Audit all — replace writes with proper portal │  
 │ sudo() in controllers (80×) │ controllers/main.py │ potential behavior │ rules │  
 │ │ │ change │ │
├─────────────────────────────┼───────────────────────────────────────────────────────┼──────────────────────────┼──────────────────────────────────────────────────┤  
 │ mail.template email_from │ edi/\*.xml (9 templates) │ Emails fail silently │ Update email_from computation expressions │
└─────────────────────────────┴───────────────────────────────────────────────────────┴──────────────────────────┴──────────────────────────────────────────────────┘

---

Recommended Test Strategy Per Phase

Each phase should pass this gate before moving to the next:

# After each phase

python odoo-bin -c odoo.conf -d test_db -i odoo_marketplace --stop-after-init

# If it installs cleanly:

python odoo-bin -c odoo.conf -d test_db --test-enable --stop-after-init -i odoo_marketplace

For phases with controller changes, add an HttpCase tour:

- Phase 3: tour that registers as a seller
- Phase 4: tour that approves a seller as admin
- Phase 7: tour that creates and submits a product for approval
- Phase 13: tour that browses the marketplace landing page and seller profile  


---

Migration Effort Estimate

┌─────────────────────────┬──────────┬──────────┬────────────────────────────┐  
 │ Phase │ Risk │ Effort │ Blocking? │  
 ├─────────────────────────┼──────────┼──────────┼────────────────────────────┤
│ 0 — Infrastructure │ Low │ 1h │ Yes — blocks all │  
 ├─────────────────────────┼──────────┼──────────┼────────────────────────────┤
│ 1 — Core Identity │ High │ 1–2 days │ Yes — god node │  
 ├─────────────────────────┼──────────┼──────────┼────────────────────────────┤  
 │ 2 — Configuration │ Medium │ 4h │ Yes — config gates │  
 ├─────────────────────────┼──────────┼──────────┼────────────────────────────┤  
 │ 3 — Pending Seller │ Medium │ 4h │ Yes — entry flow │
├─────────────────────────┼──────────┼──────────┼────────────────────────────┤  
 │ 4 — Seller Approval │ Medium │ 4h │ Yes — prerequisite gate │
├─────────────────────────┼──────────┼──────────┼────────────────────────────┤  
 │ 5 — Menu/Action Control │ Low │ 2h │ No │
├─────────────────────────┼──────────┼──────────┼────────────────────────────┤  
 │ 6 — Seller Shop │ Medium │ 4h │ No │
├─────────────────────────┼──────────┼──────────┼────────────────────────────┤  
 │ 7 — Product Management │ High │ 2 days │ Yes — feeds orders │
├─────────────────────────┼──────────┼──────────┼────────────────────────────┤  
 │ 8 — Order Management │ High │ 1–2 days │ Yes — feeds stock/payments │
├─────────────────────────┼──────────┼──────────┼────────────────────────────┤  
 │ 9 — Inventory/Stock │ High │ 1 day │ No (parallel w/ Phase 10) │
├─────────────────────────┼──────────┼──────────┼────────────────────────────┤  
 │ 10 — Payment Management │ Critical │ 2 days │ No (parallel w/ Phase 9) │
├─────────────────────────┼──────────┼──────────┼────────────────────────────┤  
 │ 11 — Reviews │ Low │ 4h │ No │
├─────────────────────────┼──────────┼──────────┼────────────────────────────┤  
 │ 12 — Dashboard │ Low │ 4h │ No │
├─────────────────────────┼──────────┼──────────┼────────────────────────────┤  
 │ 13 — Web Frontend │ High │ 3 days │ No │
└─────────────────────────┴──────────┴──────────┴────────────────────────────┘

Phases 9 and 10 can be worked in parallel by two developers once Phase 8 is complete. All phases after Phase 4 can be developed in parallel branches as long as the  
 foundational test database has Phases 0–4 installed.

---

## Claude Sessions
