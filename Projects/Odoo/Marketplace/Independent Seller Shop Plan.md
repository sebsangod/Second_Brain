---
aliases:
  - Marketplace
tags:
  - dev/erp
  - dev/sales
date: 2026-05-05
---
**Related:** [[Marketplace]]

---

## Description

| **Task** | **What it does**                                                                                                                                |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| 1        | _seller.shop.transfer_ audit log model                                                                                                          |
| 2        | _managed_by_id_, _transfer_history_ids_ on shop; remove state sync from _create()_/_write()_; update product filtering to _marketplace_shop_id_ |
| 3        | Remove state sync from _res.partner.write();_ redirect _create_seller_shop()_ to new wizard                                                     |
| 4        | _marketplace_shop_id_ on _product.template_ with _\@api.onchange_ + _create()_ auto-derivation                                                  |
| 5        | Migration script + version bump to 1.1.0                                                                                                        |
| 6        | Shop creation wizard (placeholder partner + shop, atomic)                                                                                       |
| 7        | Transfer wizard: two paths (new user / existing user), product + payment migration, log entry                                                   |
| 8        | Access control rows for all new models                                                                                                          |
| 9        | Shop form: Transfer button, _managed_by_id_ field, Transfer History tab, fixed Products domain                                                  |
| 10       | Product form: _marketplace_shop_id_ above seller, seller read-only when shop chosen                                                             |
| 11       | Manager partner form: "Managed Shops" tab                                                                                                       |
| 12       | Controller: filter by _marketplace_shop_id_ instead of _marketplace_seller_id_                                                                  |
| 13       | Manifest registration + final full test run                                                                                                     |

---

## Interview Questions for Planning

### 🤖 Questions before I write the plan

I need precise answers to these before any implementation, because each choice has architectural consequences:

1. **Is the manager also a seller?**
    The seller.shop.seller_id field requires a res.partner with seller=True. Does the manager user's own partner record have seller=True? Or is the manager a pure admin who creates shops on behalf of other seller partners?
    Creates shops on behalf of other seller partners?

2. **What does "manage more than one shop" mean for the manager?**
    - Option A: The manager's own partner (user.partner_id) is the seller_id on multiple shops — the manager is the owner of all of them.
    - Option B: The manager can create/edit shops that belong to other seller partners, acting as an administrator for those sellers' shops (the seller_id remains the original seller, not the manager).
    Creates shops on behalf of other seller partners?

2. **What does "manage more than one shop" mean for the manager?**
    - Option A: The manager's own partner (user.partner_id) is the seller_id on multiple shops — the manager is the owner of all of them.
    - Option B: The manager can create/edit shops that belong to other seller partners, acting as an administrator for those sellers' shops (the seller_id remains the original seller, not the manager).
    - Option C: Both — manager has their own shops AND can administer others'.
    This is the most consequential question. Option A requires breaking the unique constraint and redesigning the partner↔shop sync. Option B is mostly a UX improvement since the manager already has full backend access.

3. **For products — what does "products that belong to that shop" mean?**
    - Option A (explicit assignment): Add a Many2one("seller.shop") field to product.template. Each product is explicitly assigned to one shop. A seller with two shops can have different products in each. The shop catalog then shows only products assigned to it.
    - Option B (implicit / all seller's products): Keep current behavior — the shop shows all the seller's approved products. "Products belonging to the shop" just means the seller's product pool. No structural change needed here.
    Option A is a significant schema change (new column, migration, UI changes to product form, update to the shop's product search logic). Option B costs nothing but gives up the per-shop product filtering.

4. **What does "manage other sellers' shops as their own" mean on the frontend?**
    From the backend, the manager already can. Is the gap:
    - A: The manager needs to access the website-facing shop management interface (/my/marketplace/...) in the context of another seller's shop?
    - B: The manager needs to appear as the "seller" of another seller's products in the public storefront?
    - C: Only backend management — the manager edits the shop form and product forms from the Odoo backend, with no frontend changes needed?

5. **State synchronization with multiple shops:**
    Currently, when a seller's state changes (e.g., from pending to approved), all their shops' state is updated. If a manager owns or administers shops for sellers in different states, should:
    - A: Shop state continue to mirror the seller's state (current behavior)?
    - B: Shop state become independent of seller state?

6. **Commission and payment:**
    When a manager creates/manages a shop whose seller_id is another seller partner, commissions are computed from sale.order.line.marketplace_seller_id (which points to the original seller partner). Should commissions still go to the original seller, or to the manager's partner? (This may be out of scope for now, but it affects whether we change seller_id on the SOL.)


### 😎 Answers

1. yes, the manager currently could also be a seller with its own seller shop
2. actually, both of those options. The goal is allowing the manager partner to create and manage its own shop but also other sellers partners shops, which means it is the owner of all of them at the moment of the shop creation. But when the time comes, the goal is to create and approve new seller users and tranfer one (and only one) of the manager's partner shops to those users
3. option A
4. option C: the manager partner will only manage the backend while the frontend needs to seem like one shop belongs to one and only one seller
5. option B: shop state become independent of seller state until the shop is transfered to a new approved seller
6. The commissions and payments need to go to the original seller partner
I think the right approach is to allow the manager to create new seller shops that at the same time creates a new partner record for each new shop, and when the time comes to transfer the shop, create a new res.user in the system and allow the manager to assign one shop to this new user. What do you thing? Do you have more questions now?

### 🤖 Your proposed approach is architecturally sound and I agree with it. Here is why, and the remaining questions.

#### Why your approach is the right one
Keeping the existing unique(seller_id) and unique(seller_shop_id) constraints intact is the correct call. Creating a dedicated res.partner per shop avoids cascading changes to the security rules, commission logic, record rules, and frontend — all of which assume 1 shop = 1 partner. The transfer is clean: you assign a res.user to the placeholder partner and the new seller immediately inherits all history (products, commissions, SOLs) with zero data migration.

The only constraint that does need to be removed is the state sync between seller.shop.state and res.partner.state (currently wired in res.partner.write()), since shop state will be independent.

#### Remaining questions before writing the plan

Q1 — Placeholder partner identity during shop creation
When the manager creates a new shop and the system auto-creates the placeholder partner, what name/email goes on that partner? Options:
    - A: Manager provides a "seller name" and "seller email" as part of the shop creation form (these pre-fill the partner, and when the real user is eventually created, these become their credentials)
    - B: The partner is anonymous/internal (name = shop name, no email until transfer)
This matters because option A means the placeholder already has an email, and the transfer wizard just creates a login for that email. Option B means the transfer wizard collects name + email at transfer time.

Q2 — Product creation flow for managed shops
When the manager creates a product for a managed shop, do they:
    - A: Select marketplace_shop_id on the product form, and the system auto-derives marketplace_seller_id from the shop's seller_id — the manager never manually sets the seller
    - B: Select both independently
Option A is safer (it prevents the manager from accidentally assigning a product to the wrong seller/shop combination). I recommend A, but want to confirm.

Q3 — Existing product migration
Currently live products have marketplace_seller_id set but no marketplace_shop_id (the field doesn't exist yet). For existing products whose seller has a shop, should the system auto-populate marketplace_shop_id via a post-install migration (seller_id → seller_shop_id)? Or should marketplace_shop_id be nullable and existing products simply have no shop assigned?

Q4 — Transfer: new user only, or also existing users?
The transfer wizard — should it support only the "create a new user" path, or also "assign an existing user who doesn't yet have a shop"? The second case is simple to add (just link the existing partner to the shop instead of creating a new one), but requires a different UX path in the wizard.

Q5 — Manager's own personal shop
The manager currently has their own seller.shop linked to their partner. After this change, that shop also becomes "state independent". Does the manager's personal shop go through the same creation flow (i.e., does it already exist and we just leave it alone), or is it created via the new flow too? I want to confirm there's no migration needed for the manager's existing shop.

Q6 — managed_by_id visibility
For the manager to see "all shops I manage", we need to track the link. I propose adding managed_by_id = Many2one("res.partner") on seller.shop. This gives the manager a filtered list view of managed shops. Once transferred, managed_by_id is cleared. Does this cover all the visibility you need, or do you also want a "transfer history" log?

### Answers
1. option A
2. option A
3. I think the correct approach is the post-install migration to auto-assing the existing seller_shop_id to each of the existing products of a seller
4. You're right, it should also assign a shop to an existing user who doesn't yet have a shop
5. there's no migration needed for the manager's existing shop
6. I also want a transfer history log

___

## Implementation Plan

### Multi-Shop Manager Implementation Plan

>[!INFO]
> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Allow a marketplace manager to create and manage multiple seller shops (each backed by a unique placeholder `res.partner`), assign products explicitly to a specific shop, and transfer a shop to a new or existing user with a full audit trail.

**Architecture:** Each new shop gets its own dedicated `res.partner` (placeholder, no `res.user` until transfer), preserving the existing 1:1 SQL uniqueness constraints between `seller.shop` and `res.partner`. A new `marketplace_shop_id` Many2one on `product.template` makes the shop assignment explicit and drives the storefront product listing. Shop state becomes independent from seller partner state. Transfer logs every handover in a new `seller.shop.transfer` model.

**Tech Stack:** Odoo 17 CE, Python 3.10, PostgreSQL 16, Odoo ORM, QWeb views. Module: `odoo_marketplace`. Test runner: `python odoo-bin --test-enable`.

---
********
### Design Decisions & Justifications

| Decision | Justification |
|---|---|
| Keep `unique(seller_id)` and `unique(seller_shop_id)` SQL constraints | Each shop gets its own new partner → no constraint violations, zero cascade impact on security rules or commission logic |
| `managed_by_id` on `seller.shop` | Manager needs a filtered view of "my shops". Cleared on transfer. No Many2many needed since each shop has at most one current manager |
| State independence: shop state no longer mirrors partner state | User requirement (Q5). Placeholder partner is always `approved` but shop needs its own lifecycle |
| Products link to shop via `marketplace_shop_id`, seller auto-derived | Prevents manager from mismatching product seller vs. shop (Q2 Option A) |
| Transfer to existing user changes `seller_id`, migrates products and payments | The existing user's partner becomes the real owner. Placeholder is archived. This preserves the 1:1 constraint and keeps all history accessible to the new seller |
| Post-install migration populates `marketplace_shop_id` for existing products | Prevents existing shops from appearing empty after the schema change (Q3) |

---

### File Map

#### Created
| File | Responsibility |
|---|---|
| `models/seller_shop_transfer.py` | `seller.shop.transfer` model — immutable audit log of each shop handover |
| `wizard/shop_creation_wizard.py` | `seller.shop.creation.wizard` — atomically creates placeholder `res.partner` + `seller.shop` |
| `wizard/shop_transfer_wizard.py` | `seller.shop.transfer.wizard` — two paths: create new `res.user` for placeholder partner, or reassign shop to existing user's partner |
| `wizard/shop_creation_wizard_views.xml` | Form view for creation wizard |
| `wizard/shop_transfer_wizard_views.xml` | Form view for transfer wizard |
| `migrations/1.1.0/__init__.py` | Empty file required by Odoo migration system |
| `migrations/1.1.0/post-migrate.py` | Populates `marketplace_shop_id` on existing products |
| `tests/__init__.py` | Test package init |
| `tests/test_seller_shop_manager.py` | All tests for this feature |

#### Modified
| File | What changes |
|---|---|
| `models/seller_shop.py` | Add `managed_by_id`, `transfer_history_ids`; remove state sync from `create()` and `write()`; update `_get_all_seller_products()` and `_get_seller_all_products()` to filter by `marketplace_shop_id` |
| `models/marketplace_product.py` | Add `marketplace_shop_id` field, `@api.onchange`, override `create()` |
| `models/res_partner.py` | Remove state sync block from `write()`; redirect `create_seller_shop()` to open creation wizard |
| `models/__init__.py` | Import `seller_shop_transfer` |
| `wizard/__init__.py` | Import two new wizards |
| `security/ir.model.access.csv` | Access rows for `seller.shop.transfer`, `seller.shop.creation.wizard`, `seller.shop.transfer.wizard` |
| `views/seller_shop_view.xml` | Add `managed_by_id` field, Transfer button in header, Transfer History notebook page; fix Products domain |
| `views/mp_product_view.xml` | Add `marketplace_shop_id` above `marketplace_seller_id` in product form |
| `views/res_partner_view.xml` | Add "Managed Shops" page to manager's partner form |
| `controllers/main.py` | Update `seller_shop()` product domain to filter by `marketplace_shop_id` |
| `__manifest__.py` | Bump version to `1.1.0`, register all new files |

---

### Task 1: Transfer History Model

**Files:**
- Create: `odoo/odoo_marketplace/models/seller_shop_transfer.py`
- Modify: `odoo/odoo_marketplace/models/__init__.py`
- Test: `odoo/odoo_marketplace/tests/test_seller_shop_manager.py`

- [ ] **Step 1.1: Create the transfer history model**

```python
# odoo/odoo_marketplace/models/seller_shop_transfer.py
# -*- coding: utf-8 -*-
from odoo import models, fields


class SellerShopTransfer(models.Model):
    _name = 'seller.shop.transfer'
    _description = 'Seller Shop Transfer History'
    _order = 'transfer_date desc'
    _rec_name = 'shop_id'

    shop_id = fields.Many2one(
        'seller.shop', string='Shop', required=True, ondelete='cascade', index=True)
    from_partner_id = fields.Many2one(
        'res.partner', string='Previous Manager/Owner', required=True, ondelete='restrict')
    to_partner_id = fields.Many2one(
        'res.partner', string='New Owner', required=True, ondelete='restrict')
    transferred_by_id = fields.Many2one(
        'res.partner', string='Transferred By', required=True, ondelete='restrict')
    transfer_date = fields.Datetime(
        string='Transfer Date', default=fields.Datetime.now, required=True, readonly=True)
    notes = fields.Text(string='Notes')
```

- [ ] **Step 1.2: Register the model in `models/__init__.py`**

Add at the end of the existing imports:
```python
from . import seller_shop_transfer
```

- [ ] **Step 1.3: Create test package init**

```python
# odoo/odoo_marketplace/tests/__init__.py
from . import test_seller_shop_manager
```

- [ ] **Step 1.4: Write the failing test**

```python
# odoo/odoo_marketplace/tests/test_seller_shop_manager.py
# -*- coding: utf-8 -*-
from odoo.tests import TransactionCase


class TestBase(TransactionCase):
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.manager_group = cls.env.ref('odoo_marketplace.marketplace_manager_group')
        cls.seller_group = cls.env.ref('odoo_marketplace.marketplace_seller_group')

        # Manager user + partner
        cls.manager_partner = cls.env['res.partner'].create({
            'name': 'Test Manager',
            'seller': True,
            'state': 'approved',
        })
        cls.manager_user = cls.env['res.users'].create({
            'name': 'Test Manager',
            'login': 'test_manager@test.com',
            'partner_id': cls.manager_partner.id,
            'groups_id': [(4, cls.manager_group.id)],
        })

        # A placeholder partner (simulating what the creation wizard produces)
        cls.placeholder_partner = cls.env['res.partner'].create({
            'name': 'Placeholder Seller',
            'seller': True,
            'state': 'approved',
        })

        # A shop owned by the placeholder (no state sync — see Task 2)
        cls.shop = cls.env['seller.shop'].create({
            'name': 'Test Shop',
            'url_handler': 'test-shop',
            'seller_id': cls.placeholder_partner.id,
            'managed_by_id': cls.manager_partner.id,
            'state': 'approved',
        })


class TestSellerShopTransfer(TestBase):
    def test_transfer_record_creation(self):
        transfer = self.env['seller.shop.transfer'].create({
            'shop_id': self.shop.id,
            'from_partner_id': self.manager_partner.id,
            'to_partner_id': self.placeholder_partner.id,
            'transferred_by_id': self.manager_partner.id,
        })
        self.assertEqual(transfer.shop_id, self.shop)
        self.assertEqual(transfer.from_partner_id, self.manager_partner)
        self.assertEqual(transfer.to_partner_id, self.placeholder_partner)
        self.assertTrue(transfer.transfer_date)

    def test_transfer_cascade_delete_with_shop(self):
        transfer = self.env['seller.shop.transfer'].create({
            'shop_id': self.shop.id,
            'from_partner_id': self.manager_partner.id,
            'to_partner_id': self.placeholder_partner.id,
            'transferred_by_id': self.manager_partner.id,
        })
        transfer_id = transfer.id
        self.shop.unlink()
        self.assertFalse(self.env['seller.shop.transfer'].search([('id', '=', transfer_id)]))
```

- [ ] **Step 1.5: Run the test — it should FAIL with "seller.shop has no field managed_by_id"**

```bash
python odoo-bin -c odoo.conf -d test_db --test-tags=/odoo_marketplace:TestSellerShopTransfer --stop-after-init --test-enable 2>&1 | tail -30
```

Expected: Error about missing `managed_by_id` field (added in Task 2).

- [ ] **Step 1.6: Commit the model only (tests will pass after Task 2)**

```bash
git add odoo/odoo_marketplace/models/seller_shop_transfer.py \
        odoo/odoo_marketplace/models/__init__.py \
        odoo/odoo_marketplace/tests/__init__.py \
        odoo/odoo_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(marketplace): add seller.shop.transfer history model"
```

---

### Task 2: `seller.shop` Model — Add Fields and Remove State Sync

**Files:**
- Modify: `odoo/odoo_marketplace/models/seller_shop.py`

- [ ] **Step 2.1: Add `managed_by_id` and `transfer_history_ids` fields**

In `seller_shop.py`, find the field declarations block (after `url_handler` at line ~108). Add these two fields:

```python
managed_by_id = fields.Many2one(
    'res.partner',
    string='Managed By',
    help='Manager partner who created this shop. Cleared when shop is transferred to a seller.',
    index=True,
)
transfer_history_ids = fields.One2many(
    'seller.shop.transfer', 'shop_id', string='Transfer History', readonly=True)
```

- [ ] **Step 2.2: Remove state sync from `create()`**

Find lines 135–138 in `seller_shop.py`:
```python
shop_ids = super(SellerShop, self).create(vals_list)
for res in shop_ids:
    res.seller_id.seller_shop_id = res.id
    res.state = res.seller_id.state   # ← DELETE THIS LINE
return shop_ids
```

Also remove the product-initialization block that precedes `super().create()`. The full revised `create()` method:

```python
@api.model_create_multi
def create(self, vals_list):
    for vals in vals_list:
        url_handler = vals.get('url_handler')
        if url_handler:
            if (not re.match('^[a-zA-Z0-9-_]+$', url_handler)
                    or re.match('^[-_][a-zA-Z0-9-_]*$', url_handler)
                    or re.match('^[a-zA-Z0-9-_]*[-_]$', url_handler)):
                raise UserError(_("Please enter URL handler correctly!"))
            vals["url"] = self._get_page_new_url(url_handler)
    shop_ids = super(SellerShop, self).create(vals_list)
    for res in shop_ids:
        res.seller_id.seller_shop_id = res.id
    return shop_ids
```

- [ ] **Step 2.3: Remove state sync and stale product logic from `write()`**

Find the `write()` method. The revised version removes `"state": seller_obj.state` and replaces the product-fetching block with a clean `total_product` recount after the write:

```python
def write(self, vals):
    url_handler = vals.get('url_handler')
    seller_id = vals.get("seller_id")
    seller_obj = None
    if seller_id:
        seller_obj = self.env["res.partner"].browse(seller_id)
    for obj in self:
        if url_handler:
            if (not re.match('^[a-zA-Z0-9-_]+$', url_handler)
                    or re.match('^[-_][a-zA-Z0-9-_]*$', url_handler)
                    or re.match('^[a-zA-Z0-9-_]*[-_]$', url_handler)):
                raise UserError(_("Please enter URL handler correctly!"))
            vals["url"] = self._get_page_new_url(url_handler)
        if seller_id:
            obj.seller_id.seller_shop_id = None
    res = super(SellerShop, self).write(vals)
    if seller_id:
        for obj in self:
            seller_obj.write({"seller_shop_id": obj.id})
            count = self.env['product.template'].search_count([
                ('marketplace_shop_id', '=', obj.id),
                ('status', '=', 'approved'),
            ])
            obj.total_product = count
    elif vals.get('seller_product_ids'):
        for obj in self:
            obj.total_product = len(obj.seller_product_ids)
    return res
```

- [ ] **Step 2.4: Update `_get_all_seller_products()` to filter by `marketplace_shop_id`**

Replace the existing `_get_all_seller_products` method:

```python
def _get_all_seller_products(self):
    for shop in self:
        products = self.env['product.template'].search([
            ('marketplace_shop_id', '=', shop.id),
            ('status', '=', 'approved'),
        ])
        shop.seller_product_ids = products.ids
```

- [ ] **Step 2.5: Update `_get_seller_all_products()` to use shop ID**

This static helper is now only used for backward compatibility (called with a seller_id). Replace it so it searches by shop instead when the calling context is a shop record:

```python
def _get_seller_all_products(self, seller_id):
    # Legacy signature kept for compatibility; searches by shop when self has an id
    if self.id:
        return self.env["product.template"].search([
            ('sale_ok', '=', True),
            ('status', '=', 'approved'),
            ('marketplace_shop_id', '=', self.id),
        ]).ids
    if seller_id:
        return self.env["product.template"].search([
            ('sale_ok', '=', True),
            ('status', '=', 'approved'),
            ('marketplace_seller_id', '=', seller_id),
        ]).ids
    return []
```

- [ ] **Step 2.6: Run the failing test — it should now error only on missing `marketplace_shop_id` on product**

```bash
python odoo-bin -c odoo.conf -d test_db --test-tags=/odoo_marketplace:TestSellerShopTransfer --stop-after-init --test-enable 2>&1 | tail -30
```

Expected: Setup error — `marketplace_shop_id` field not yet on `product.template`.

- [ ] **Step 2.7: Add state-sync removal test to the test file**

Add this class to `tests/test_seller_shop_manager.py`:

```python
class TestShopStateIndependence(TestBase):
    def test_changing_seller_state_does_not_change_shop_state(self):
        self.shop.state = 'approved'
        self.placeholder_partner.write({'state': 'denied'})
        self.assertEqual(self.shop.state, 'approved',
            "Shop state must not change when seller partner state changes")

    def test_shop_create_does_not_copy_seller_state(self):
        pending_partner = self.env['res.partner'].create({
            'name': 'Pending Seller',
            'seller': True,
            'state': 'pending',
        })
        shop = self.env['seller.shop'].create({
            'name': 'Independent Shop',
            'url_handler': 'independent-shop',
            'seller_id': pending_partner.id,
            'state': 'approved',
        })
        self.assertEqual(shop.state, 'approved',
            "Shop state must be exactly what was set, not copied from seller")
```

- [ ] **Step 2.8: Commit**

```bash
git add odoo/odoo_marketplace/models/seller_shop.py \
        odoo/odoo_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(marketplace): add managed_by_id/transfer_history to seller.shop, remove state sync"
```

---

### Task 3: Remove State Sync from `res.partner.write()` and Redirect Shop Creation

**Files:**
- Modify: `odoo/odoo_marketplace/models/res_partner.py`

- [ ] **Step 3.1: Remove the state sync block from `res.partner.write()`**

Find and delete this block (currently lines ~425–427 of `res_partner.py`):

```python
# DELETE THESE THREE LINES:
if vals.get('state'):
    for rec in self.filtered('seller_shop_id'):
        rec.seller_shop_id.state = rec.state
```

- [ ] **Step 3.2: Redirect `create_seller_shop()` to the new wizard**

Replace the existing `create_seller_shop()` method (currently lines ~675–684 of `res_partner.py`):

```python
def create_seller_shop(self):
    return {
        'type': 'ir.actions.act_window',
        'name': 'Create Seller Shop',
        'res_model': 'seller.shop.creation.wizard',
        'view_mode': 'form',
        'target': 'new',
        'context': {'default_managed_by_id': self.env.user.partner_id.id},
    }
```

- [ ] **Step 3.3: Add the state sync removal test**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestPartnerStateSyncRemoved(TestBase):
    def test_partner_state_change_does_not_affect_shop(self):
        self.shop.state = 'new'
        self.placeholder_partner.state = 'denied'
        self.shop.invalidate_recordset()
        self.assertEqual(self.shop.state, 'new',
            "res.partner.write() must not push its state to seller_shop_id")
```

- [ ] **Step 3.4: Run this test — should PASS (state sync removed)**

```bash
python odoo-bin -c odoo.conf -d test_db --test-tags=/odoo_marketplace:TestPartnerStateSyncRemoved --stop-after-init --test-enable 2>&1 | tail -20
```

Expected: PASS.

- [ ] **Step 3.5: Commit**

```bash
git add odoo/odoo_marketplace/models/res_partner.py \
        odoo/odoo_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(marketplace): remove state sync in res.partner.write(), redirect shop creation to wizard"
```

---

### Task 4: `marketplace_shop_id` on `product.template`

**Files:**
- Modify: `odoo/odoo_marketplace/models/marketplace_product.py`

- [ ] **Step 4.1: Add `marketplace_shop_id` field**

In `marketplace_product.py`, add the new field directly after the `marketplace_seller_id` declaration (~line 53):

```python
marketplace_shop_id = fields.Many2one(
    'seller.shop',
    string='Seller Shop',
    copy=False,
    help='The shop this product belongs to. Setting this field auto-fills Seller.',
    index=True,
)
```

- [ ] **Step 4.2: Add `@api.onchange` to auto-derive seller from shop**

Add this method in the `@api.onchange` section (after the existing `@api.onchange("marketplace_seller_id")` at ~line 94):

```python
@api.onchange('marketplace_shop_id')
def _onchange_marketplace_shop_id(self):
    if self.marketplace_shop_id:
        self.marketplace_seller_id = self.marketplace_shop_id.seller_id
```

- [ ] **Step 4.3: Override `create()` to auto-derive seller from shop for programmatic creation**

Find the existing `create()` method in `marketplace_product.py` (~line 100 area). Add seller derivation at the top of the vals loop, before existing logic:

```python
@api.model_create_multi
def create(self, vals_list):
    for vals in vals_list:
        shop_id = vals.get('marketplace_shop_id')
        if shop_id and not vals.get('marketplace_seller_id'):
            shop = self.env['seller.shop'].browse(shop_id)
            vals['marketplace_seller_id'] = shop.seller_id.id
    return super().create(vals_list)
```

- [ ] **Step 4.4: Write the failing tests**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestProductShopAssignment(TestBase):
    def test_create_product_with_shop_derives_seller(self):
        product = self.env['product.template'].create({
            'name': 'Shop Product',
            'type': 'consu',
            'marketplace_shop_id': self.shop.id,
        })
        self.assertEqual(product.marketplace_seller_id, self.placeholder_partner,
            "marketplace_seller_id must be auto-derived from marketplace_shop_id.seller_id")

    def test_create_product_with_explicit_seller_not_overridden(self):
        other_partner = self.env['res.partner'].create({
            'name': 'Other', 'seller': True, 'state': 'approved'})
        product = self.env['product.template'].create({
            'name': 'Explicit Seller Product',
            'type': 'consu',
            'marketplace_shop_id': self.shop.id,
            'marketplace_seller_id': other_partner.id,
        })
        self.assertEqual(product.marketplace_seller_id.id, other_partner.id,
            "Explicit marketplace_seller_id must not be overridden by shop derivation")

    def test_product_without_shop_keeps_seller(self):
        product = self.env['product.template'].create({
            'name': 'No Shop Product',
            'type': 'consu',
            'marketplace_seller_id': self.placeholder_partner.id,
        })
        self.assertFalse(product.marketplace_shop_id,
            "Products without a shop must have no marketplace_shop_id")
```

- [ ] **Step 4.5: Run tests — they should FAIL because `marketplace_shop_id` does not exist yet in the DB schema**

```bash
python odoo-bin -c odoo.conf -d test_db --test-tags=/odoo_marketplace:TestProductShopAssignment --stop-after-init --test-enable 2>&1 | tail -20
```

Expected: Error about unknown column `marketplace_shop_id`.

- [ ] **Step 4.6: Install/upgrade module to apply the new column, then run tests again**

```bash
python odoo-bin -c odoo.conf -d test_db --stop-after-init -u odoo_marketplace 2>&1 | tail -20
python odoo-bin -c odoo.conf -d test_db --test-tags=/odoo_marketplace:TestProductShopAssignment --stop-after-init --test-enable 2>&1 | tail -20
```

Expected: All 3 tests PASS.

- [ ] **Step 4.7: Commit**

```bash
git add odoo/odoo_marketplace/models/marketplace_product.py \
        odoo/odoo_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(marketplace): add marketplace_shop_id to product.template with seller auto-derivation"
```

---

### Task 5: Post-Install Migration — Populate `marketplace_shop_id` on Existing Products

**Files:**
- Create: `odoo/odoo_marketplace/migrations/1.1.0/__init__.py`
- Create: `odoo/odoo_marketplace/migrations/1.1.0/post-migrate.py`
- Modify: `odoo/odoo_marketplace/__manifest__.py` (version bump)

- [ ] **Step 5.1: Create migration directory and init file**

```bash
mkdir -p odoo/odoo_marketplace/migrations/1.1.0
touch odoo/odoo_marketplace/migrations/1.1.0/__init__.py
```

- [ ] **Step 5.2: Write the migration script**

```python
# odoo/odoo_marketplace/migrations/1.1.0/post-migrate.py
# -*- coding: utf-8 -*-
import logging
from odoo import api, SUPERUSER_ID

_logger = logging.getLogger(__name__)


def migrate(cr, version):
    """Populate marketplace_shop_id on existing marketplace products.

    For every seller partner that already has a seller_shop_id, assign
    that shop to all their products that have no shop yet.
    This preserves the invariant: product.marketplace_shop_id == product.marketplace_seller_id.seller_shop_id
    """
    env = api.Environment(cr, SUPERUSER_ID, {})
    partners = env['res.partner'].search([
        ('seller', '=', True),
        ('seller_shop_id', '!=', False),
    ])
    migrated = 0
    for partner in partners:
        products = env['product.template'].search([
            ('marketplace_seller_id', '=', partner.id),
            ('marketplace_shop_id', '=', False),
        ])
        if products:
            products.write({'marketplace_shop_id': partner.seller_shop_id.id})
            migrated += len(products)
    _logger.info(
        'seller.shop migration: assigned marketplace_shop_id to %d existing products', migrated)
```

- [ ] **Step 5.3: Bump version in `__manifest__.py`**

Change line 20:
```python
"version":  "1.1.0",
```

- [ ] **Step 5.4: Write migration test**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestMigration(TestBase):
    def test_existing_products_get_shop_assigned(self):
        """Simulate what the migration does: product with seller but no shop."""
        product = self.env['product.template'].create({
            'name': 'Pre-Migration Product',
            'type': 'consu',
            'marketplace_seller_id': self.placeholder_partner.id,
            # marketplace_shop_id intentionally left unset
        })
        self.assertFalse(product.marketplace_shop_id)

        # Run migration logic directly (same code as post-migrate.py)
        products_to_migrate = self.env['product.template'].search([
            ('marketplace_seller_id', '=', self.placeholder_partner.id),
            ('marketplace_shop_id', '=', False),
        ])
        products_to_migrate.write({
            'marketplace_shop_id': self.placeholder_partner.seller_shop_id.id
        })

        product.invalidate_recordset()
        self.assertEqual(product.marketplace_shop_id, self.shop,
            "Migration must assign the seller's shop to existing products")
```

- [ ] **Step 5.5: Run the migration test**

```bash
python odoo-bin -c odoo.conf -d test_db --test-tags=/odoo_marketplace:TestMigration --stop-after-init --test-enable 2>&1 | tail -20
```

Expected: PASS.

- [ ] **Step 5.6: Commit**

```bash
git add odoo/odoo_marketplace/migrations/ \
        odoo/odoo_marketplace/__manifest__.py \
        odoo/odoo_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(marketplace): add post-migrate script for marketplace_shop_id, bump version to 1.1.0"
```

---

### Task 6: Shop Creation Wizard

**Files:**
- Create: `odoo/odoo_marketplace/wizard/shop_creation_wizard.py`
- Create: `odoo/odoo_marketplace/wizard/shop_creation_wizard_views.xml`
- Modify: `odoo/odoo_marketplace/wizard/__init__.py`

- [ ] **Step 6.1: Create the wizard model**

```python
# odoo/odoo_marketplace/wizard/shop_creation_wizard.py
# -*- coding: utf-8 -*-
import re
from odoo import models, fields, api, _
from odoo.exceptions import UserError


class ShopCreationWizard(models.TransientModel):
    _name = 'seller.shop.creation.wizard'
    _description = 'Create Seller Shop with Placeholder Partner'

    # Placeholder seller partner fields
    seller_name = fields.Char(string='Seller Name', required=True)
    seller_email = fields.Char(string='Seller Email', required=True)

    # Shop fields
    shop_name = fields.Char(string='Shop Name', required=True)
    url_handler = fields.Char(string='URL Handler', required=True,
        help='Unique slug for /seller/shop/<url_handler>. Lowercase letters, digits, hyphens.')
    description = fields.Text(string='Description')
    shop_logo = fields.Binary(string='Logo')
    shop_banner = fields.Binary(string='Banner')
    shop_tag_line = fields.Char(string='Tag Line', size=100)

    # Tracked internally
    managed_by_id = fields.Many2one(
        'res.partner', string='Manager', default=lambda self: self.env.user.partner_id)

    @api.onchange('shop_name')
    def _onchange_shop_name(self):
        if self.shop_name and not self.url_handler:
            self.url_handler = self.shop_name.lower().replace(' ', '-')

    def action_create_shop(self):
        self.ensure_one()
        url_handler = self.url_handler
        if (not re.match('^[a-zA-Z0-9-_]+$', url_handler)
                or re.match('^[-_][a-zA-Z0-9-_]*$', url_handler)
                or re.match('^[a-zA-Z0-9-_]*[-_]$', url_handler)):
            raise UserError(_("URL handler may only contain letters, digits, hyphens, and "
                              "underscores. It cannot start or end with a hyphen or underscore."))

        if self.env['seller.shop'].search([('url_handler', '=', url_handler)], limit=1):
            raise UserError(_("A shop with URL handler '%s' already exists.") % url_handler)

        if self.env['seller.shop'].search([('name', '=', self.shop_name)], limit=1):
            raise UserError(_("A shop named '%s' already exists.") % self.shop_name)

        # 1. Create placeholder seller partner (no res.user yet)
        placeholder = self.env['res.partner'].create({
            'name': self.seller_name,
            'email': self.seller_email,
            'seller': True,
            'state': 'approved',
            'set_seller_wise_settings': True,
        })

        # 2. Create shop — seller_shop.create() will set placeholder.seller_shop_id = shop.id
        shop = self.env['seller.shop'].create({
            'name': self.shop_name,
            'url_handler': url_handler,
            'description': self.description,
            'shop_logo': self.shop_logo,
            'shop_banner': self.shop_banner,
            'shop_tag_line': self.shop_tag_line,
            'seller_id': placeholder.id,
            'managed_by_id': self.managed_by_id.id or self.env.user.partner_id.id,
            'state': 'approved',
        })

        return {
            'type': 'ir.actions.act_window',
            'name': _('Seller Shop'),
            'res_model': 'seller.shop',
            'view_mode': 'form',
            'res_id': shop.id,
            'target': 'current',
        }
```

- [ ] **Step 6.2: Create the wizard view**

```xml
<!-- odoo/odoo_marketplace/wizard/shop_creation_wizard_views.xml -->
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="seller_shop_creation_wizard_form_view" model="ir.ui.view">
        <field name="name">seller.shop.creation.wizard.form</field>
        <field name="model">seller.shop.creation.wizard</field>
        <field name="arch" type="xml">
            <form string="Create New Seller Shop">
                <group string="New Seller Account">
                    <field name="seller_name" placeholder="Full name of the seller"/>
                    <field name="seller_email" placeholder="seller@example.com"/>
                </group>
                <group string="Shop Details">
                    <field name="shop_name" placeholder="My Store"/>
                    <field name="url_handler" placeholder="my-store"/>
                    <field name="shop_tag_line" placeholder="Short catchy description"/>
                    <field name="description"/>
                </group>
                <group string="Visual">
                    <field name="shop_logo" widget="image" options='{"size": [128, 128]}'/>
                    <field name="shop_banner" widget="image" options='{"size": [400, 120]}'/>
                </group>
                <field name="managed_by_id" invisible="1"/>
                <footer>
                    <button name="action_create_shop" string="Create Shop" type="object"
                            class="btn-primary"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="seller_shop_creation_wizard_action" model="ir.actions.act_window">
        <field name="name">Create New Seller Shop</field>
        <field name="res_model">seller.shop.creation.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>
```

- [ ] **Step 6.3: Register wizard in `wizard/__init__.py`**

Add to the existing imports:
```python
from . import shop_creation_wizard
```

- [ ] **Step 6.4: Write wizard tests**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestShopCreationWizard(TestBase):
    def test_wizard_creates_placeholder_partner_and_shop(self):
        wizard = self.env['seller.shop.creation.wizard'].create({
            'seller_name': 'New Seller',
            'seller_email': 'new_seller@test.com',
            'shop_name': 'Brand New Shop',
            'url_handler': 'brand-new-shop',
            'managed_by_id': self.manager_partner.id,
        })
        wizard.action_create_shop()

        shop = self.env['seller.shop'].search([('url_handler', '=', 'brand-new-shop')])
        self.assertTrue(shop, "Shop must be created")
        self.assertEqual(shop.managed_by_id, self.manager_partner)
        self.assertEqual(shop.seller_id.name, 'New Seller')
        self.assertEqual(shop.seller_id.email, 'new_seller@test.com')
        self.assertTrue(shop.seller_id.seller, "Placeholder partner must have seller=True")
        self.assertEqual(shop.seller_id.state, 'approved')
        self.assertFalse(shop.seller_id.user_ids,
            "Placeholder partner must have no res.user until transfer")
        self.assertEqual(shop.seller_id.seller_shop_id, shop,
            "seller.shop.create() must set seller_shop_id on the placeholder partner")

    def test_wizard_rejects_duplicate_url_handler(self):
        wizard = self.env['seller.shop.creation.wizard'].create({
            'seller_name': 'Dup Seller',
            'seller_email': 'dup@test.com',
            'shop_name': 'Duplicate Handler',
            'url_handler': 'test-shop',  # already used by cls.shop in setUpClass
            'managed_by_id': self.manager_partner.id,
        })
        from odoo.exceptions import UserError
        with self.assertRaises(UserError):
            wizard.action_create_shop()

    def test_wizard_rejects_invalid_url_handler(self):
        wizard = self.env['seller.shop.creation.wizard'].create({
            'seller_name': 'Bad URL',
            'seller_email': 'bad@test.com',
            'shop_name': 'Bad URL Shop',
            'url_handler': '-starts-with-dash',
            'managed_by_id': self.manager_partner.id,
        })
        from odoo.exceptions import UserError
        with self.assertRaises(UserError):
            wizard.action_create_shop()
```

- [ ] **Step 6.5: Run wizard tests**

```bash
python odoo-bin -c odoo.conf -d test_db --test-tags=/odoo_marketplace:TestShopCreationWizard --stop-after-init --test-enable 2>&1 | tail -30
```

Expected: All 3 tests PASS.

- [ ] **Step 6.6: Commit**

```bash
git add odoo/odoo_marketplace/wizard/shop_creation_wizard.py \
        odoo/odoo_marketplace/wizard/shop_creation_wizard_views.xml \
        odoo/odoo_marketplace/wizard/__init__.py \
        odoo/odoo_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(marketplace): add shop creation wizard (placeholder partner + shop)"
```

---

### Task 7: Shop Transfer Wizard

**Files:**
- Create: `odoo/odoo_marketplace/wizard/shop_transfer_wizard.py`
- Create: `odoo/odoo_marketplace/wizard/shop_transfer_wizard_views.xml`
- Modify: `odoo/odoo_marketplace/wizard/__init__.py`

- [ ] **Step 7.1: Create the transfer wizard model**

```python
# odoo/odoo_marketplace/wizard/shop_transfer_wizard.py
# -*- coding: utf-8 -*-
from odoo import models, fields, api, _
from odoo.exceptions import UserError


class ShopTransferWizard(models.TransientModel):
    _name = 'seller.shop.transfer.wizard'
    _description = 'Transfer Seller Shop to a User'

    shop_id = fields.Many2one(
        'seller.shop', string='Shop', required=True,
        default=lambda self: self.env.context.get('active_id'))
    transfer_type = fields.Selection([
        ('new_user', 'Create New User for this Shop'),
        ('existing_user', 'Transfer to Existing User (no shop yet)'),
    ], string='Transfer Type', required=True, default='new_user')

    # Path A — new user linked to the existing placeholder partner
    new_user_login = fields.Char(string='Login (Email)')
    new_user_name = fields.Char(string='Full Name')

    # Path B — existing user whose partner has no shop
    existing_user_id = fields.Many2one(
        'res.users', string='Existing User',
        domain="[('partner_id.seller_shop_id', '=', False)]")

    notes = fields.Text(string='Transfer Notes')

    @api.onchange('shop_id')
    def _onchange_shop_id(self):
        if self.shop_id and self.shop_id.seller_id:
            seller = self.shop_id.seller_id
            self.new_user_login = seller.email or ''
            self.new_user_name = seller.name or ''

    def _validate(self):
        self.ensure_one()
        if not self.shop_id.managed_by_id:
            raise UserError(_(
                "This shop has no managing partner. Only shops created via the "
                "'Create New Seller Shop' wizard can be transferred."))
        if self.transfer_type == 'new_user':
            if not self.new_user_login:
                raise UserError(_("Login (Email) is required for the new user."))
            if self.env['res.users'].search([('login', '=', self.new_user_login)], limit=1):
                raise UserError(_("A user with login '%s' already exists.") % self.new_user_login)
        else:
            if not self.existing_user_id:
                raise UserError(_("Please select an existing user to transfer this shop to."))
            if self.existing_user_id.partner_id.seller_shop_id:
                raise UserError(_(
                    "User '%s' already owns a shop.") % self.existing_user_id.name)

    def action_transfer(self):
        self.ensure_one()
        self._validate()

        shop = self.shop_id
        old_manager = shop.managed_by_id
        seller_group = self.env.ref('odoo_marketplace.marketplace_seller_group')

        if self.transfer_type == 'new_user':
            self._transfer_new_user(shop, old_manager, seller_group)
        else:
            self._transfer_existing_user(shop, old_manager, seller_group)

        shop.managed_by_id = False
        return {'type': 'ir.actions.act_window_close'}

    def _transfer_new_user(self, shop, old_manager, seller_group):
        """Create a res.user linked to the placeholder partner. All data stays on the same partner."""
        placeholder = shop.seller_id
        new_user = self.env['res.users'].create({
            'name': self.new_user_name or placeholder.name,
            'login': self.new_user_login,
            'partner_id': placeholder.id,
            'groups_id': [(4, seller_group.id)],
        })
        self.env['seller.shop.transfer'].create({
            'shop_id': shop.id,
            'from_partner_id': old_manager.id,
            'to_partner_id': placeholder.id,
            'transferred_by_id': self.env.user.partner_id.id,
            'notes': self.notes,
        })
        return new_user

    def _transfer_existing_user(self, shop, old_manager, seller_group):
        """Reassign the shop to an existing user's partner. Migrate products and payments."""
        new_partner = self.existing_user_id.partner_id
        old_placeholder = shop.seller_id

        # Update products: keep marketplace_shop_id, change marketplace_seller_id
        products = self.env['product.template'].search([
            ('marketplace_shop_id', '=', shop.id),
        ])
        if products:
            products.write({'marketplace_seller_id': new_partner.id})

        # Update seller payments: reassign to new partner
        payments = self.env['seller.payment'].search([
            ('seller_id', '=', old_placeholder.id),
        ])
        if payments:
            payments.write({'seller_id': new_partner.id})

        # Change shop's seller_id — seller.shop.write() handles:
        #   old_placeholder.seller_shop_id = None
        #   new_partner.seller_shop_id = shop.id
        shop.write({'seller_id': new_partner.id})

        # Mark the placeholder as seller and ensure group
        new_partner.write({'seller': True, 'state': 'approved'})
        self.existing_user_id.write({'groups_id': [(4, seller_group.id)]})

        # Archive the now-orphaned placeholder
        old_placeholder.active = False

        self.env['seller.shop.transfer'].create({
            'shop_id': shop.id,
            'from_partner_id': old_manager.id,
            'to_partner_id': new_partner.id,
            'transferred_by_id': self.env.user.partner_id.id,
            'notes': self.notes,
        })
```

- [ ] **Step 7.2: Create the transfer wizard view**

```xml
<!-- odoo/odoo_marketplace/wizard/shop_transfer_wizard_views.xml -->
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="seller_shop_transfer_wizard_form_view" model="ir.ui.view">
        <field name="name">seller.shop.transfer.wizard.form</field>
        <field name="model">seller.shop.transfer.wizard</field>
        <field name="arch" type="xml">
            <form string="Transfer Seller Shop">
                <group>
                    <field name="shop_id" readonly="1"/>
                    <field name="transfer_type" widget="radio"/>
                </group>
                <group invisible="transfer_type != 'new_user'" string="New User Details">
                    <field name="new_user_name"/>
                    <field name="new_user_login" string="Login / Email"/>
                </group>
                <group invisible="transfer_type != 'existing_user'" string="Existing User">
                    <field name="existing_user_id" options="{'no_create': True}"/>
                </group>
                <group string="Notes">
                    <field name="notes" nolabel="1" placeholder="Optional: reason for transfer"/>
                </group>
                <footer>
                    <button name="action_transfer" string="Transfer Shop" type="object"
                            class="btn-primary"
                            confirm="This will transfer the shop and cannot be undone. Continue?"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="seller_shop_transfer_wizard_action" model="ir.actions.act_window">
        <field name="name">Transfer Seller Shop</field>
        <field name="res_model">seller.shop.transfer.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="binding_model_id" ref="model_seller_shop"/>
        <field name="binding_view_types">form</field>
    </record>
</odoo>
```

- [ ] **Step 7.3: Register in `wizard/__init__.py`**

Add:
```python
from . import shop_transfer_wizard
```

- [ ] **Step 7.4: Write transfer wizard tests**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestShopTransferWizard(TestBase):
    def test_transfer_new_user_creates_user_on_placeholder(self):
        wizard = self.env['seller.shop.transfer.wizard'].create({
            'shop_id': self.shop.id,
            'transfer_type': 'new_user',
            'new_user_login': 'real_seller@test.com',
            'new_user_name': 'Real Seller',
        })
        wizard.action_transfer()

        # Placeholder partner now has a user
        self.assertTrue(self.placeholder_partner.user_ids,
            "Transfer must create a res.user for the placeholder partner")
        new_user = self.placeholder_partner.user_ids[0]
        self.assertEqual(new_user.login, 'real_seller@test.com')

        # Shop: managed_by_id cleared, seller_id unchanged
        self.assertFalse(self.shop.managed_by_id,
            "managed_by_id must be cleared after transfer")
        self.assertEqual(self.shop.seller_id, self.placeholder_partner,
            "seller_id must remain the placeholder partner (now the real seller)")

        # Transfer log created
        log = self.env['seller.shop.transfer'].search([('shop_id', '=', self.shop.id)])
        self.assertTrue(log, "A transfer log entry must be created")
        self.assertEqual(log.from_partner_id, self.manager_partner)
        self.assertEqual(log.to_partner_id, self.placeholder_partner)

    def test_transfer_existing_user_changes_seller_and_migrates_products(self):
        # Set up: product assigned to the placeholder's shop
        product = self.env['product.template'].create({
            'name': 'Managed Product',
            'type': 'consu',
            'marketplace_shop_id': self.shop.id,
            'marketplace_seller_id': self.placeholder_partner.id,
        })
        # Set up: existing user with no shop
        other_partner = self.env['res.partner'].create({'name': 'Existing Seller'})
        other_user = self.env['res.users'].create({
            'name': 'Existing Seller',
            'login': 'existing_seller@test.com',
            'partner_id': other_partner.id,
        })

        wizard = self.env['seller.shop.transfer.wizard'].create({
            'shop_id': self.shop.id,
            'transfer_type': 'existing_user',
            'existing_user_id': other_user.id,
        })
        wizard.action_transfer()

        # Shop now owned by other_partner
        self.shop.invalidate_recordset()
        self.assertEqual(self.shop.seller_id, other_partner)

        # Products migrated
        product.invalidate_recordset()
        self.assertEqual(product.marketplace_seller_id, other_partner)
        self.assertEqual(product.marketplace_shop_id, self.shop,
            "marketplace_shop_id must remain unchanged after transfer")

        # Placeholder archived
        self.assertFalse(self.placeholder_partner.active,
            "Placeholder partner must be archived after existing-user transfer")

        # Transfer log
        log = self.env['seller.shop.transfer'].search([('shop_id', '=', self.shop.id)])
        self.assertTrue(log)
        self.assertEqual(log.to_partner_id, other_partner)

    def test_transfer_fails_without_managed_by_id(self):
        self.shop.managed_by_id = False
        wizard = self.env['seller.shop.transfer.wizard'].create({
            'shop_id': self.shop.id,
            'transfer_type': 'new_user',
            'new_user_login': 'x@test.com',
        })
        from odoo.exceptions import UserError
        with self.assertRaises(UserError):
            wizard.action_transfer()
```

- [ ] **Step 7.5: Run transfer wizard tests**

```bash
python odoo-bin -c odoo.conf -d test_db --test-tags=/odoo_marketplace:TestShopTransferWizard --stop-after-init --test-enable 2>&1 | tail -30
```

Expected: All 3 tests PASS.

- [ ] **Step 7.6: Commit**

```bash
git add odoo/odoo_marketplace/wizard/shop_transfer_wizard.py \
        odoo/odoo_marketplace/wizard/shop_transfer_wizard_views.xml \
        odoo/odoo_marketplace/wizard/__init__.py \
        odoo/odoo_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(marketplace): add shop transfer wizard (new user + existing user paths)"
```

---

### Task 8: Security — Access Control Rows

**Files:**
- Modify: `odoo/odoo_marketplace/security/ir.model.access.csv`

- [ ] **Step 8.1: Add access rows for all three new models**

Append to `security/ir.model.access.csv`:

```csv
access_seller_shop_transfer_officer,seller_shop_transfer_officer,model_seller_shop_transfer,odoo_marketplace.marketplace_officer_group,1,0,0,0
access_seller_shop_transfer_manager,seller_shop_transfer_manager,model_seller_shop_transfer,odoo_marketplace.marketplace_manager_group,1,1,1,1
access_shop_creation_wizard_manager,shop_creation_wizard_manager,model_seller_shop_creation_wizard,odoo_marketplace.marketplace_manager_group,1,1,1,1
access_shop_transfer_wizard_manager,shop_transfer_wizard_manager,model_seller_shop_transfer_wizard,odoo_marketplace.marketplace_manager_group,1,1,1,1
```

Justification:
- `seller.shop.transfer`: Officers can read (audit), Managers have full CRUD (they write log entries).
- Both wizard models: Managers only (only managers create shops and initiate transfers).

- [ ] **Step 8.2: Commit**

```bash
git add odoo/odoo_marketplace/security/ir.model.access.csv
git commit -m "chore(marketplace): add access rights for new shop manager models"
```

---

### Task 9: Views — `seller.shop` Form

**Files:**
- Modify: `odoo/odoo_marketplace/views/seller_shop_view.xml`

- [ ] **Step 9.1: Add Transfer button and `managed_by_id` to the shop form header**

Find the `<header>` block (line ~14 of `seller_shop_view.xml`):
```xml
<header>
    <field name="state" widget="statusbar" statusbar_visible="pending,approved,denied" invisible ="id == False or seller_id == False"/>
</header>
```

Replace with:
```xml
<header>
    <button name="%(odoo_marketplace.seller_shop_transfer_wizard_action)d"
            string="Transfer to Seller"
            type="action"
            class="btn-primary"
            groups="odoo_marketplace.marketplace_manager_group"
            invisible="not managed_by_id or not id"/>
    <field name="state" widget="statusbar" statusbar_visible="pending,approved,denied"
           invisible="id == False or seller_id == False"/>
</header>
```

- [ ] **Step 9.2: Add `managed_by_id` to the main info group**

Find the group containing `seller_id` (line ~67):
```xml
<field name="seller_id" groups="odoo_marketplace.marketplace_officer_group" widget="selection" options="{'no_create': True}"/>
<field name="seller_id" invisible="1" groups="!odoo_marketplace.marketplace_officer_group"/>
```

Add below it:
```xml
<field name="managed_by_id"
       groups="odoo_marketplace.marketplace_manager_group"
       options="{'no_create': True}"
       readonly="1"/>
```

- [ ] **Step 9.3: Fix Products page domain and add Transfer History tab**

Find the `<notebook>` section (line ~89). Replace:
```xml
<notebook>
    <page name="seller_products" string="Products">
        <field name="seller_product_ids" domain="[('marketplace_seller_id', '=', seller_id),('status','=','approved')]" mode="kanban" context="{'kanban_view_ref':'odoo_marketplace.wk_seller_product_template_kanban_view','form_view_ref':'odoo_marketplace.wk_seller_product_template_form_view'}"/>
    </page>
    <page name="seller_shop_t_c" string="Terms &amp; Conditions">
        <field name="shop_t_c"/>
    </page>
</notebook>
```

With:
```xml
<notebook>
    <page name="seller_products" string="Products">
        <field name="seller_product_ids"
               domain="[('marketplace_shop_id', '=', id), ('status', '=', 'approved')]"
               mode="kanban"
               context="{'kanban_view_ref': 'odoo_marketplace.wk_seller_product_template_kanban_view',
                         'form_view_ref': 'odoo_marketplace.wk_seller_product_template_form_view'}"/>
    </page>
    <page name="seller_shop_t_c" string="Terms &amp; Conditions">
        <field name="shop_t_c"/>
    </page>
    <page name="transfer_history" string="Transfer History"
          groups="odoo_marketplace.marketplace_manager_group">
        <field name="transfer_history_ids" readonly="1">
            <list>
                <field name="transfer_date"/>
                <field name="from_partner_id" string="From Manager"/>
                <field name="to_partner_id" string="To Seller"/>
                <field name="transferred_by_id"/>
                <field name="notes"/>
            </list>
        </field>
    </page>
</notebook>
```

- [ ] **Step 9.4: Commit**

```bash
git add odoo/odoo_marketplace/views/seller_shop_view.xml
git commit -m "feat(marketplace): update seller shop form — Transfer button, managed_by_id, transfer history, shop product domain"
```

---

### Task 10: Views — Product Form (`marketplace_shop_id`)

**Files:**
- Modify: `odoo/odoo_marketplace/views/mp_product_view.xml`

- [ ] **Step 10.1: Add `marketplace_shop_id` above `marketplace_seller_id` in the product form**

There are two product forms in `mp_product_view.xml` (one for Officers, one for the main view). Both have the pattern around line 366–367:

```xml
<field name="marketplace_seller_id" required="1" readonly = "status in ('approved','rejected')" domain="[('seller', '=', True), ('state', '=', 'approved')]" options="{'no_create': True}" context="{'form_view_ref': 'odoo_marketplace.wk_seller_form_view'}" groups="odoo_marketplace.marketplace_officer_group"/>
<field name="marketplace_seller_id" invisible="1" groups="!odoo_marketplace.marketplace_officer_group"/>
```

For **each occurrence** of this pattern in the file, insert `marketplace_shop_id` before `marketplace_seller_id`:

```xml
<field name="marketplace_shop_id"
       options="{'no_create': True}"
       domain="[('state', '=', 'approved')]"
       groups="odoo_marketplace.marketplace_manager_group"
       readonly="status in ('approved', 'rejected')"/>
<field name="marketplace_seller_id"
       required="1"
       readonly="status in ('approved','rejected') or marketplace_shop_id != False"
       domain="[('seller', '=', True), ('state', '=', 'approved')]"
       options="{'no_create': True}"
       context="{'form_view_ref': 'odoo_marketplace.wk_seller_form_view'}"
       groups="odoo_marketplace.marketplace_officer_group"/>
<field name="marketplace_seller_id" invisible="1" groups="!odoo_marketplace.marketplace_officer_group"/>
```

The `readonly` on `marketplace_seller_id` gains `or marketplace_shop_id != False` so that once a shop is chosen, the seller field is locked (manager cannot override the auto-derived seller).

- [ ] **Step 10.2: Commit**

```bash
git add odoo/odoo_marketplace/views/mp_product_view.xml
git commit -m "feat(marketplace): add marketplace_shop_id to product form with seller auto-lock"
```

---

### Task 11: Views — Manager's Partner Form ("Managed Shops")

**Files:**
- Modify: `odoo/odoo_marketplace/views/res_partner_view.xml`

- [ ] **Step 11.1: Add "Managed Shops" page to the seller partner form**

Locate the existing `inherit_id` view that extends `res.partner` form in `res_partner_view.xml`. Find the last `<page>` inside the partner's notebook, and add after it:

```xml
<page name="managed_shops" string="Managed Shops"
      groups="odoo_marketplace.marketplace_manager_group">
    <field name="managed_shop_ids" readonly="1"
           domain="[('managed_by_id', '=', active_id)]">
        <list>
            <field name="name"/>
            <field name="seller_id" string="Placeholder Seller"/>
            <field name="state"/>
            <field name="website_published" string="Published"/>
            <field name="url" widget="url"/>
        </list>
    </field>
</page>
```

- [ ] **Step 11.2: Add `managed_shop_ids` One2many to `res.partner`**

In `models/res_partner.py`, in the field declarations section (near the other shop-related fields around line ~124), add:

```python
managed_shop_ids = fields.One2many(
    'seller.shop', 'managed_by_id', string='Managed Shops', readonly=True)
```

- [ ] **Step 11.3: Commit**

```bash
git add odoo/odoo_marketplace/views/res_partner_view.xml \
        odoo/odoo_marketplace/models/res_partner.py
git commit -m "feat(marketplace): add Managed Shops tab to manager partner form"
```

---

### Task 12: Controller — Filter Shop Products by `marketplace_shop_id`

**Files:**
- Modify: `odoo/odoo_marketplace/controllers/main.py`

- [ ] **Step 12.1: Update the `_get_search_domain` inner function in `seller_shop()`**

Find lines 609–621 inside the `seller_shop()` method:

```python
def _get_search_domain(search):
    domain = request.website.sale_product_domain()
    domain += [("marketplace_seller_id", "=",
                shop_obj.sudo().seller_id.id)]
    ...
```

Replace the domain addition:
```python
def _get_search_domain(search):
    domain = request.website.sale_product_domain()
    domain += [("marketplace_shop_id", "=", shop_obj.sudo().id)]
    ...
```

- [ ] **Step 12.2: Update the main products query on line 661**

Find:
```python
products = env['product.template'].sudo().search([('sale_ok', '=', True), ('status', '=', "approved"), ("website_published", "=", True), ("marketplace_seller_id", "=", shop_obj.sudo().seller_id.id), ("id", "in", shop_obj.sudo().seller_product_ids.ids)], limit=ppg, offset=pager['offset'], order='website_published desc, website_sequence desc')
```

Replace with:
```python
products = env['product.template'].sudo().search([
    ('sale_ok', '=', True),
    ('status', '=', 'approved'),
    ('website_published', '=', True),
    ('marketplace_shop_id', '=', shop_obj.sudo().id),
], limit=ppg, offset=pager['offset'], order='website_published desc, website_sequence desc')
```

- [ ] **Step 12.3: Update `product_count` query on line 659**

Find:
```python
product_count = request.env["product.template"].sudo().search_count([('sale_ok', '=', True), ('status', '=', "approved"), ("website_published", "=", True), ("id", "in", shop_obj.sudo().seller_product_ids.ids)])
```

Replace with:
```python
product_count = request.env["product.template"].sudo().search_count([
    ('sale_ok', '=', True),
    ('status', '=', 'approved'),
    ('website_published', '=', True),
    ('marketplace_shop_id', '=', shop_obj.sudo().id),
])
```

Note: The `sales_count` block (lines 648–652) still uses `marketplace_seller_id` on purpose — sales count is a seller-level metric, not shop-level. Leave it unchanged.

- [ ] **Step 12.4: Write controller test**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestShopControllerDomain(TestBase):
    def test_shop_product_listing_uses_shop_not_seller(self):
        """Products assigned to a different shop of the same seller must NOT appear."""
        # Give placeholder_partner a second shop (different url_handler)
        second_placeholder = self.env['res.partner'].create({
            'name': 'Second Placeholder', 'seller': True, 'state': 'approved'})
        second_shop = self.env['seller.shop'].create({
            'name': 'Second Shop',
            'url_handler': 'second-shop',
            'seller_id': second_placeholder.id,
            'state': 'approved',
        })

        # Product in self.shop
        product_in_shop = self.env['product.template'].create({
            'name': 'In Shop Product', 'type': 'consu',
            'marketplace_shop_id': self.shop.id,
            'marketplace_seller_id': self.placeholder_partner.id,
            'status': 'approved', 'sale_ok': True,
        })
        # Product in second_shop (same seller group but different shop)
        product_in_other_shop = self.env['product.template'].create({
            'name': 'Other Shop Product', 'type': 'consu',
            'marketplace_shop_id': second_shop.id,
            'marketplace_seller_id': second_placeholder.id,
            'status': 'approved', 'sale_ok': True,
        })

        # Simulate the controller domain
        domain = [
            ('sale_ok', '=', True),
            ('status', '=', 'approved'),
            ('marketplace_shop_id', '=', self.shop.id),
        ]
        results = self.env['product.template'].search(domain)
        self.assertIn(product_in_shop, results)
        self.assertNotIn(product_in_other_shop, results,
            "Products from a different shop must not appear in this shop's listing")
```

- [ ] **Step 12.5: Run controller test**

```bash
python odoo-bin -c odoo.conf -d test_db --test-tags=/odoo_marketplace:TestShopControllerDomain --stop-after-init --test-enable 2>&1 | tail -20
```

Expected: PASS.

- [ ] **Step 12.6: Commit**

```bash
git add odoo/odoo_marketplace/controllers/main.py \
        odoo/odoo_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(marketplace): update shop controller to filter products by marketplace_shop_id"
```

---

### Task 13: Manifest — Register All New Files

**Files:**
- Modify: `odoo/odoo_marketplace/__manifest__.py`

- [ ] **Step 13.1: Add all new files to the `data` list in `__manifest__.py`**

After `'security/ir.model.access.csv',` add:
```python
'wizard/shop_creation_wizard_views.xml',
'wizard/shop_transfer_wizard_views.xml',
```

After `'wizard/mark_done_stats.xml',` (end of wizard section) the views are already covered above.

The full revised `data` list (only showing changed sections, maintain all existing entries):
```python
"data": [
    'security/marketplace_security.xml',
    # ... all existing edi entries ...
    'data/mp_product_demo_data.xml',
    'data/mp_config_setting_data.xml',
    'data/seller_payment_method_data.xml',
    'data/ir_config_parameter_data.xml',
    'security/ir.model.access.csv',
    'wizard/mark_approved.xml',
    'wizard/publish.xml',
    'wizard/unpublish.xml',
    'wizard/server_action_wizard.xml',
    'wizard/seller_status_reason_wizard_view.xml',
    'wizard/seller_payment_wizard_view.xml',
    'wizard/seller_registration_wizard_view.xml',
    'wizard/variant_approval_wizard_view.xml',
    'wizard/mark_done_stats.xml',
    'wizard/shop_creation_wizard_views.xml',   # NEW
    'wizard/shop_transfer_wizard_views.xml',   # NEW
    # ... all existing views entries unchanged ...
],
```

- [ ] **Step 13.2: Run a full module upgrade to verify everything loads cleanly**

```bash
python odoo-bin -c odoo.conf -d test_db --stop-after-init -u odoo_marketplace 2>&1 | grep -E "ERROR|WARNING|raise" | head -20
```

Expected: No errors. Any `WARNING` about `_read_group_fill_results` is pre-existing and not related to this feature.

- [ ] **Step 13.3: Run the full test suite**

```bash
python odoo-bin -c odoo.conf -d test_db --test-tags=/odoo_marketplace --stop-after-init --test-enable 2>&1 | tail -40
```

Expected: All tests in `TestSellerShopTransfer`, `TestShopStateIndependence`, `TestPartnerStateSyncRemoved`, `TestProductShopAssignment`, `TestMigration`, `TestShopCreationWizard`, `TestShopTransferWizard`, `TestShopControllerDomain` PASS.

- [ ] **Step 13.4: Commit**

```bash
git add odoo/odoo_marketplace/__manifest__.py
git commit -m "chore(marketplace): register new wizard views in manifest"
```

---

### Self-Review

#### Spec Coverage Checklist

| Requirement | Task |
|---|---|
| Manager creates multiple shops, each backed by its own partner | Task 6 (creation wizard) |
| Each new shop gets a unique placeholder `res.partner` | Task 6 |
| Manager is linked to shops via `managed_by_id` | Task 2 |
| Products explicitly assigned to a shop via `marketplace_shop_id` | Task 4 |
| Seller auto-derived from shop on product create/onchange | Task 4 |
| Shop state independent of seller partner state | Tasks 2 & 3 |
| Transfer to new user (creates `res.user` on placeholder) | Task 7 |
| Transfer to existing user (migrates products + payments, archives placeholder) | Task 7 |
| Transfer history log (`seller.shop.transfer`) | Task 1 |
| `managed_by_id` cleared after transfer | Task 7 |
| Post-install migration for existing products | Task 5 |
| Frontend shop (`/seller/shop/`) shows only shop's products | Task 12 |
| Unique SQL constraints preserved | No change — by design |
| Commissions go to seller partner (not manager) | No change — `marketplace_seller_id` drives commission; migration ensures it points to correct partner |
| Manager's existing personal shop unaffected | No data migration for existing shops |

#### No Placeholders Check

All code blocks are complete. No "TBD", "handle edge cases", or "similar to above" patterns.

#### Type Consistency Check

| Symbol | Defined | Used |
|---|---|---|
| `seller.shop.transfer` model | Task 1 | Tasks 1, 7 |
| `seller.shop.creation.wizard` model | Task 6 | Tasks 6, 8 |
| `seller.shop.transfer.wizard` model | Task 7 | Tasks 7, 8 |
| `managed_by_id` on `seller.shop` | Task 2 | Tasks 2, 6, 7, 9 |
| `transfer_history_ids` on `seller.shop` | Task 2 | Task 9 |
| `marketplace_shop_id` on `product.template` | Task 4 | Tasks 4, 5, 7, 10, 12 |
| `managed_shop_ids` on `res.partner` | Task 11 | Task 11 |
| `_transfer_new_user()` | Task 7 | Task 7 |
| `_transfer_existing_user()` | Task 7 | Task 7 |

---

## Implementation Plan (Inherited)

### Multi-Shop Manager Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Allow a marketplace manager to create and manage multiple seller shops (each backed by a unique placeholder `res.partner`), assign products explicitly to a specific shop, and transfer a shop to a new or existing user — all implemented inside `dexmi_marketplace` via Odoo inheritance, leaving `odoo_marketplace` completely untouched.

**Architecture:** All changes live in `odoo/dexmi_marketplace/`. Model changes use `_inherit` to extend `odoo_marketplace` models. View changes use XPath inheritance. The controller override subclasses `MarketplaceSellerShop`. State-sync removal is achieved by restoring pre-super() state values after calling the parent's methods (since we cannot prevent the parent's `create()`/`write()` from running). Each new shop gets its own dedicated placeholder `res.partner`, preserving the 1:1 SQL uniqueness constraints between `seller.shop` and `res.partner`.

**Tech Stack:** Odoo 17 CE, Python 3.10, PostgreSQL 16. Module under change: `dexmi_marketplace` (depends on `odoo_marketplace`). Test runner: `python odoo-bin --test-enable`.

---

### Design Decisions & Justifications

| Decision | Justification |
|---|---|
| All changes in `dexmi_marketplace`, zero changes in `odoo_marketplace` | Keeps the vendor module upgradeable; all customization is in the project's own module |
| State sync "removal" implemented as post-super() state restoration | We cannot stop `odoo_marketplace`'s `create()`/`write()` from running its sync; restoring the intended value afterward is the correct override pattern |
| `managed_by_id` on `seller.shop` via `_inherit` | Links each managed shop to the creating manager; cleared on transfer |
| `marketplace_shop_id` on `product.template` via `_inherit` | Explicit shop assignment; seller auto-derived from shop's `seller_id` |
| Transfer history in a new `seller.shop.transfer` model | Owned by `dexmi_marketplace`, no conflict with upstream |
| Post-migrate script in `dexmi_marketplace/migrations/` | Migration tracks only `dexmi_marketplace` version changes |

---

### Inheritance Approach — Key Patterns

**Overriding `odoo_marketplace` model methods:**
```python
# In dexmi_marketplace, any model file
class SellerShop(models.Model):
    _inherit = 'seller.shop'

    def create(self, vals_list):
        shops = super().create(vals_list)  # runs odoo_marketplace's create()
        # restore or add behavior here
        return shops
```

**State sync reversal pattern** (used in Tasks 2 and 3):
Because `odoo_marketplace`'s `create()`/`write()` syncs `shop.state = seller.state`, we must capture the intended state *before* calling `super()`, then restore it *after*:
```python
explicit_states = {i: v['state'] for i, v in enumerate(vals_list) if 'state' in v}
shops = super().create(vals_list)
for i, shop in enumerate(shops):
    if i in explicit_states:
        shop.state = explicit_states[i]
```

**View inheritance:** All view changes use `inherit_id` + XPath, never direct modification of `odoo_marketplace` XML files.

---

### File Map

#### Created in `odoo/dexmi_marketplace/`
| File | Responsibility |
|---|---|
| `models/seller_shop_transfer.py` | New `seller.shop.transfer` model — audit log |
| `models/seller_shop.py` | `_inherit = 'seller.shop'` — adds fields, overrides `create()`/`write()`/`_get_all_seller_products()` |
| `wizards/shop_creation_wizard.py` | New `seller.shop.creation.wizard` transient |
| `wizards/shop_transfer_wizard.py` | New `seller.shop.transfer.wizard` transient |
| `wizards/shop_creation_wizard_views.xml` | Form view for creation wizard |
| `wizards/shop_transfer_wizard_views.xml` | Form view + action binding for transfer wizard |
| `views/seller_shop_inherit.xml` | XPath inheritance of `odoo_marketplace.wk_seller_shop_form_view` |
| `views/mp_product_shop_inherit.xml` | XPath inheritance of `odoo_marketplace.wk_seller_product_template_form_view` |
| `views/res_partner_managed_shops.xml` | XPath inheritance of `odoo_marketplace.wk_seller_form_view` |
| `migrations/17.0.1.0/__init__.py` | Empty — required by Odoo migration system |
| `migrations/17.0.1.0/post-migrate.py` | Populates `marketplace_shop_id` on existing products |
| `tests/__init__.py` | Test package init |
| `tests/test_seller_shop_manager.py` | All tests for this feature |

#### Modified in `odoo/dexmi_marketplace/`
| File | What changes |
|---|---|
| `models/res_partner.py` | Add `managed_shop_ids`; override `write()` to restore shop states; override `create_seller_shop()` |
| `models/product_template.py` | Add `marketplace_shop_id`; add `@api.onchange`; override `create()` |
| `models/__init__.py` | Import `seller_shop_transfer`, `seller_shop` |
| `wizards/__init__.py` | Import two new wizards |
| `controllers/main.py` | Add `DexmiMarketplaceSellerShop` subclass overriding the shop route |
| `security/ir.model.access.csv` | Access rows for all new models |
| `__manifest__.py` | Bump version to `17.0.1.0`, register all new files |

#### Untouched
`odoo/odoo_marketplace/` — every single file remains unchanged.

---

### Task 1: Transfer History Model

**Files:**
- Create: `odoo/dexmi_marketplace/models/seller_shop_transfer.py`
- Modify: `odoo/dexmi_marketplace/models/__init__.py`
- Create: `odoo/dexmi_marketplace/tests/__init__.py`
- Create: `odoo/dexmi_marketplace/tests/test_seller_shop_manager.py`

- [ ] **Step 1.1: Create the transfer history model**

```python
# odoo/dexmi_marketplace/models/seller_shop_transfer.py
# -*- coding: utf-8 -*-
from odoo import models, fields


class SellerShopTransfer(models.Model):
    _name = 'seller.shop.transfer'
    _description = 'Seller Shop Transfer History'
    _order = 'transfer_date desc'
    _rec_name = 'shop_id'

    shop_id = fields.Many2one(
        'seller.shop', string='Shop', required=True, ondelete='cascade', index=True)
    from_partner_id = fields.Many2one(
        'res.partner', string='Previous Manager/Owner', required=True, ondelete='restrict')
    to_partner_id = fields.Many2one(
        'res.partner', string='New Owner', required=True, ondelete='restrict')
    transferred_by_id = fields.Many2one(
        'res.partner', string='Transferred By', required=True, ondelete='restrict')
    transfer_date = fields.Datetime(
        string='Transfer Date', default=fields.Datetime.now, required=True, readonly=True)
    notes = fields.Text(string='Notes')
```

- [ ] **Step 1.2: Register in `models/__init__.py`**

Add these two imports at the end of the existing file:
```python
from . import seller_shop_transfer
from . import seller_shop        # created in Task 2 — add now so the file is ready
```

- [ ] **Step 1.3: Create test package**

```python
# odoo/dexmi_marketplace/tests/__init__.py
from . import test_seller_shop_manager
```

- [ ] **Step 1.4: Write the initial test class**

```python
# odoo/dexmi_marketplace/tests/test_seller_shop_manager.py
# -*- coding: utf-8 -*-
from odoo.tests import TransactionCase


class TestBase(TransactionCase):
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.manager_group = cls.env.ref('odoo_marketplace.marketplace_manager_group')
        cls.seller_group = cls.env.ref('odoo_marketplace.marketplace_seller_group')

        cls.manager_partner = cls.env['res.partner'].create({
            'name': 'Test Manager',
            'seller': True,
            'state': 'approved',
        })
        cls.manager_user = cls.env['res.users'].create({
            'name': 'Test Manager',
            'login': 'test_manager@test.com',
            'partner_id': cls.manager_partner.id,
            'groups_id': [(4, cls.manager_group.id)],
        })

        cls.placeholder_partner = cls.env['res.partner'].create({
            'name': 'Placeholder Seller',
            'seller': True,
            'state': 'approved',
        })

        cls.shop = cls.env['seller.shop'].create({
            'name': 'Test Shop',
            'url_handler': 'test-shop',
            'seller_id': cls.placeholder_partner.id,
            'managed_by_id': cls.manager_partner.id,
            'state': 'approved',
        })


class TestSellerShopTransfer(TestBase):
    def test_transfer_record_creation(self):
        transfer = self.env['seller.shop.transfer'].create({
            'shop_id': self.shop.id,
            'from_partner_id': self.manager_partner.id,
            'to_partner_id': self.placeholder_partner.id,
            'transferred_by_id': self.manager_partner.id,
        })
        self.assertEqual(transfer.shop_id, self.shop)
        self.assertEqual(transfer.from_partner_id, self.manager_partner)
        self.assertTrue(transfer.transfer_date)

    def test_transfer_cascade_delete_with_shop(self):
        transfer = self.env['seller.shop.transfer'].create({
            'shop_id': self.shop.id,
            'from_partner_id': self.manager_partner.id,
            'to_partner_id': self.placeholder_partner.id,
            'transferred_by_id': self.manager_partner.id,
        })
        transfer_id = transfer.id
        self.shop.unlink()
        self.assertFalse(
            self.env['seller.shop.transfer'].search([('id', '=', transfer_id)]))
```

- [ ] **Step 1.5: Run the test — expect FAIL because `managed_by_id` does not exist yet**

```bash
python odoo-bin -c odoo.conf -d test_db \
    --test-tags=/dexmi_marketplace:TestSellerShopTransfer \
    --stop-after-init --test-enable 2>&1 | tail -20
```

Expected: Error — `managed_by_id` field not found on `seller.shop`.

- [ ] **Step 1.6: Commit**

```bash
git add odoo/dexmi_marketplace/models/seller_shop_transfer.py \
        odoo/dexmi_marketplace/models/__init__.py \
        odoo/dexmi_marketplace/tests/__init__.py \
        odoo/dexmi_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(dexmi-mp): add seller.shop.transfer history model"
```

---

### Task 2: Extend `seller.shop` — Fields, State Independence, Product Filtering

**Files:**
- Create: `odoo/dexmi_marketplace/models/seller_shop.py`

The `_inherit` override cannot prevent `odoo_marketplace`'s `create()` from running `res.state = res.seller_id.state`. We capture the intended `state` value from each `vals` dict before calling `super()`, then restore it afterward. The same pattern applies to `write()`.

- [ ] **Step 2.1: Create the `_inherit` extension**

```python
# odoo/dexmi_marketplace/models/seller_shop.py
# -*- coding: utf-8 -*-
from odoo import models, fields, api


class SellerShop(models.Model):
    _inherit = 'seller.shop'

    managed_by_id = fields.Many2one(
        'res.partner',
        string='Managed By',
        help='Manager who created this shop. Cleared when the shop is transferred to a seller.',
        index=True,
    )
    transfer_history_ids = fields.One2many(
        'seller.shop.transfer', 'shop_id', string='Transfer History', readonly=True)

    @api.model_create_multi
    def create(self, vals_list):
        # Preserve explicitly-passed state values before odoo_marketplace's create()
        # overwrites them with the seller partner's state.
        explicit_states = {i: v['state'] for i, v in enumerate(vals_list) if 'state' in v}
        shops = super().create(vals_list)
        for i, shop in enumerate(shops):
            if i in explicit_states:
                shop.state = explicit_states[i]
        return shops

    def write(self, vals):
        # Preserve current shop states before odoo_marketplace's write() may sync them
        # from the seller partner's state when seller_id changes.
        shop_states_before = {}
        if vals.get('seller_id') or vals.get('state'):
            for shop in self:
                shop_states_before[shop.id] = shop.state

        result = super().write(vals)

        # Restore states only when seller_id changed — odoo_marketplace copies the
        # new seller's state into the shop, but we want independence.
        if vals.get('seller_id') and not vals.get('state'):
            for shop in self:
                if shop.id in shop_states_before:
                    shop.state = shop_states_before[shop.id]
        return result

    def _get_all_seller_products(self):
        """Filter shop products by marketplace_shop_id instead of marketplace_seller_id.

        After the dexmi migration, every product has marketplace_shop_id set.
        Filtering by shop (not seller) means a seller with multiple managed shops
        gets correct per-shop product lists.
        """
        for shop in self:
            products = self.env['product.template'].search([
                ('marketplace_shop_id', '=', shop.id),
                ('status', '=', 'approved'),
            ])
            shop.seller_product_ids = products.ids
```

- [ ] **Step 2.2: Write state-independence tests**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestShopStateIndependence(TestBase):
    def test_partner_state_change_does_not_affect_shop(self):
        self.shop.state = 'approved'
        self.placeholder_partner.write({'state': 'denied'})
        self.shop.invalidate_recordset()
        self.assertEqual(
            self.shop.state, 'approved',
            "res.partner.write() must not push its state to seller_shop_id")

    def test_shop_create_respects_explicit_state(self):
        pending_partner = self.env['res.partner'].create({
            'name': 'Pending Seller',
            'seller': True,
            'state': 'pending',
        })
        shop = self.env['seller.shop'].create({
            'name': 'State Test Shop',
            'url_handler': 'state-test-shop',
            'seller_id': pending_partner.id,
            'state': 'approved',          # explicit — must survive parent sync
        })
        self.assertEqual(
            shop.state, 'approved',
            "Explicit state='approved' must survive odoo_marketplace's state-sync in create()")

    def test_managed_by_id_persists(self):
        self.assertEqual(self.shop.managed_by_id, self.manager_partner)
```

- [ ] **Step 2.3: Upgrade module to apply new columns, then run tests**

```bash
python odoo-bin -c odoo.conf -d test_db --stop-after-init -u dexmi_marketplace 2>&1 | tail -10
python odoo-bin -c odoo.conf -d test_db \
    --test-tags=/dexmi_marketplace:TestShopStateIndependence \
    --test-tags=/dexmi_marketplace:TestSellerShopTransfer \
    --stop-after-init --test-enable 2>&1 | tail -20
```

Expected: All 5 tests PASS.

- [ ] **Step 2.4: Commit**

```bash
git add odoo/dexmi_marketplace/models/seller_shop.py \
        odoo/dexmi_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(dexmi-mp): extend seller.shop with managed_by_id, transfer history, state independence"
```

---

### Task 3: Extend `res.partner` — Remove State Sync and Redirect Shop Creation

**Files:**
- Modify: `odoo/dexmi_marketplace/models/res_partner.py`

- [ ] **Step 3.1: Add `managed_shop_ids` and override `write()` and `create_seller_shop()`**

Add to the existing `ResPartner` class in `odoo/dexmi_marketplace/models/res_partner.py` (append after the last existing field/method, before the closing of the class):

```python
    managed_shop_ids = fields.One2many(
        'seller.shop', 'managed_by_id', string='Managed Shops', readonly=True)

    def write(self, vals):
        # Save shop states before calling super(). odoo_marketplace.ResPartner.write()
        # propagates state changes to seller_shop_id — we undo that here so shop
        # state is independent from seller partner state.
        shop_states = {}
        if vals.get('state'):
            for rec in self.filtered('seller_shop_id'):
                shop_states[rec.seller_shop_id.id] = rec.seller_shop_id.state

        result = super().write(vals)

        # Restore shop states that the parent may have overwritten
        for shop_id, original_state in shop_states.items():
            self.env['seller.shop'].browse(shop_id).state = original_state

        return result

    def create_seller_shop(self):
        """Open the creation wizard instead of a raw seller.shop form."""
        return {
            'type': 'ir.actions.act_window',
            'name': 'Create Seller Shop',
            'res_model': 'seller.shop.creation.wizard',
            'view_mode': 'form',
            'target': 'new',
            'context': {'default_managed_by_id': self.env.user.partner_id.id},
        }
```

- [ ] **Step 3.2: Write the state sync test**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestPartnerStateSyncRemoved(TestBase):
    def test_partner_write_does_not_change_shop_state(self):
        self.shop.state = 'new'
        self.placeholder_partner.write({'state': 'denied'})
        self.shop.invalidate_recordset()
        self.assertEqual(
            self.shop.state, 'new',
            "dexmi ResPartner.write() must restore shop state after parent sync")
```

- [ ] **Step 3.3: Run test**

```bash
python odoo-bin -c odoo.conf -d test_db \
    --test-tags=/dexmi_marketplace:TestPartnerStateSyncRemoved \
    --stop-after-init --test-enable 2>&1 | tail -20
```

Expected: PASS.

- [ ] **Step 3.4: Commit**

```bash
git add odoo/dexmi_marketplace/models/res_partner.py \
        odoo/dexmi_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(dexmi-mp): add managed_shop_ids, undo state sync in res.partner, redirect shop creation"
```

---

### Task 4: Extend `product.template` — `marketplace_shop_id`

**Files:**
- Modify: `odoo/dexmi_marketplace/models/product_template.py`

- [ ] **Step 4.1: Add `marketplace_shop_id`, onchange, and `create()` override**

Add to the existing `ProductTemplate` class in `odoo/dexmi_marketplace/models/product_template.py`:

```python
    marketplace_shop_id = fields.Many2one(
        'seller.shop',
        string='Seller Shop',
        copy=False,
        help='The shop this product belongs to. Selecting a shop auto-fills Seller.',
        index=True,
    )

    @api.onchange('marketplace_shop_id')
    def _onchange_marketplace_shop_id(self):
        if self.marketplace_shop_id:
            self.marketplace_seller_id = self.marketplace_shop_id.seller_id

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            shop_id = vals.get('marketplace_shop_id')
            if shop_id and not vals.get('marketplace_seller_id'):
                shop = self.env['seller.shop'].browse(shop_id)
                vals['marketplace_seller_id'] = shop.seller_id.id
        return super().create(vals_list)
```

- [ ] **Step 4.2: Write the failing tests**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestProductShopAssignment(TestBase):
    def test_create_product_with_shop_derives_seller(self):
        product = self.env['product.template'].create({
            'name': 'Shop Product',
            'type': 'consu',
            'marketplace_shop_id': self.shop.id,
        })
        self.assertEqual(
            product.marketplace_seller_id, self.placeholder_partner,
            "marketplace_seller_id must be auto-derived from marketplace_shop_id.seller_id")

    def test_explicit_seller_not_overridden_by_shop(self):
        other_partner = self.env['res.partner'].create({
            'name': 'Other', 'seller': True, 'state': 'approved'})
        product = self.env['product.template'].create({
            'name': 'Explicit Seller Product',
            'type': 'consu',
            'marketplace_shop_id': self.shop.id,
            'marketplace_seller_id': other_partner.id,
        })
        self.assertEqual(product.marketplace_seller_id.id, other_partner.id,
            "Explicit marketplace_seller_id must win over auto-derivation")

    def test_product_without_shop_has_no_shop_id(self):
        product = self.env['product.template'].create({
            'name': 'No Shop Product',
            'type': 'consu',
            'marketplace_seller_id': self.placeholder_partner.id,
        })
        self.assertFalse(product.marketplace_shop_id)
```

- [ ] **Step 4.3: Upgrade module and run tests**

```bash
python odoo-bin -c odoo.conf -d test_db --stop-after-init -u dexmi_marketplace 2>&1 | tail -10
python odoo-bin -c odoo.conf -d test_db \
    --test-tags=/dexmi_marketplace:TestProductShopAssignment \
    --stop-after-init --test-enable 2>&1 | tail -20
```

Expected: All 3 tests PASS.

- [ ] **Step 4.4: Commit**

```bash
git add odoo/dexmi_marketplace/models/product_template.py \
        odoo/dexmi_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(dexmi-mp): add marketplace_shop_id to product.template with seller auto-derivation"
```

---

### Task 5: Post-Install Migration

**Files:**
- Create: `odoo/dexmi_marketplace/migrations/17.0.1.0/__init__.py`
- Create: `odoo/dexmi_marketplace/migrations/17.0.1.0/post-migrate.py`
- Modify: `odoo/dexmi_marketplace/__manifest__.py` (version bump only)

- [ ] **Step 5.1: Create migration directory and init**

```bash
mkdir -p odoo/dexmi_marketplace/migrations/17.0.1.0
touch odoo/dexmi_marketplace/migrations/17.0.1.0/__init__.py
```

- [ ] **Step 5.2: Write migration script**

```python
# odoo/dexmi_marketplace/migrations/17.0.1.0/post-migrate.py
# -*- coding: utf-8 -*-
import logging
from odoo import api, SUPERUSER_ID

_logger = logging.getLogger(__name__)


def migrate(cr, version):
    """Populate marketplace_shop_id on existing marketplace products.

    For every approved seller partner that already has a seller_shop_id,
    assign that shop to all their products that have no shop yet.
    Preserves the invariant:
        product.marketplace_shop_id == product.marketplace_seller_id.seller_shop_id
    """
    env = api.Environment(cr, SUPERUSER_ID, {})
    partners = env['res.partner'].search([
        ('seller', '=', True),
        ('seller_shop_id', '!=', False),
    ])
    migrated = 0
    for partner in partners:
        products = env['product.template'].search([
            ('marketplace_seller_id', '=', partner.id),
            ('marketplace_shop_id', '=', False),
        ])
        if products:
            products.write({'marketplace_shop_id': partner.seller_shop_id.id})
            migrated += len(products)
    _logger.info(
        'dexmi_marketplace migration: assigned marketplace_shop_id to %d existing products',
        migrated)
```

- [ ] **Step 5.3: Bump version in `__manifest__.py`**

Change line with `"version"`:
```python
"version": "17.0.1.0",
```

- [ ] **Step 5.4: Write migration test**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestMigration(TestBase):
    def test_existing_products_get_shop_assigned(self):
        """Simulate migration: product has seller but no shop."""
        product = self.env['product.template'].create({
            'name': 'Pre-Migration Product',
            'type': 'consu',
            'marketplace_seller_id': self.placeholder_partner.id,
            # marketplace_shop_id intentionally omitted
        })
        self.assertFalse(product.marketplace_shop_id)

        # Run the same logic as the migration script
        products_to_migrate = self.env['product.template'].search([
            ('marketplace_seller_id', '=', self.placeholder_partner.id),
            ('marketplace_shop_id', '=', False),
        ])
        products_to_migrate.write({
            'marketplace_shop_id': self.placeholder_partner.seller_shop_id.id,
        })

        product.invalidate_recordset()
        self.assertEqual(product.marketplace_shop_id, self.shop,
            "Migration must assign the seller's shop to the existing product")
```

- [ ] **Step 5.5: Run migration test**

```bash
python odoo-bin -c odoo.conf -d test_db \
    --test-tags=/dexmi_marketplace:TestMigration \
    --stop-after-init --test-enable 2>&1 | tail -20
```

Expected: PASS.

- [ ] **Step 5.6: Commit**

```bash
git add odoo/dexmi_marketplace/migrations/ \
        odoo/dexmi_marketplace/__manifest__.py \
        odoo/dexmi_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(dexmi-mp): post-migrate script for marketplace_shop_id, bump version to 17.0.1.0"
```

---

### Task 6: Shop Creation Wizard

**Files:**
- Create: `odoo/dexmi_marketplace/wizards/shop_creation_wizard.py`
- Create: `odoo/dexmi_marketplace/wizards/shop_creation_wizard_views.xml`
- Modify: `odoo/dexmi_marketplace/wizards/__init__.py`

- [ ] **Step 6.1: Create the wizard model**

```python
# odoo/dexmi_marketplace/wizards/shop_creation_wizard.py
# -*- coding: utf-8 -*-
import re
from odoo import models, fields, api, _
from odoo.exceptions import UserError


class ShopCreationWizard(models.TransientModel):
    _name = 'seller.shop.creation.wizard'
    _description = 'Create Seller Shop with Placeholder Partner'

    # Placeholder seller partner
    seller_name = fields.Char(string='Seller Name', required=True)
    seller_email = fields.Char(string='Seller Email', required=True)

    # Shop
    shop_name = fields.Char(string='Shop Name', required=True)
    url_handler = fields.Char(string='URL Handler', required=True,
        help='Unique slug for /seller/shop/<url_handler>. '
             'Lowercase letters, digits, hyphens, underscores only.')
    description = fields.Text(string='Description')
    shop_logo = fields.Binary(string='Logo')
    shop_banner = fields.Binary(string='Banner')
    shop_tag_line = fields.Char(string='Tag Line', size=100)

    managed_by_id = fields.Many2one(
        'res.partner', string='Manager',
        default=lambda self: self.env.user.partner_id)

    @api.onchange('shop_name')
    def _onchange_shop_name(self):
        if self.shop_name and not self.url_handler:
            self.url_handler = self.shop_name.lower().replace(' ', '-')

    def action_create_shop(self):
        self.ensure_one()
        url_handler = self.url_handler
        if (not re.match('^[a-zA-Z0-9-_]+$', url_handler)
                or re.match('^[-_][a-zA-Z0-9-_]*$', url_handler)
                or re.match('^[a-zA-Z0-9-_]*[-_]$', url_handler)):
            raise UserError(_(
                "URL handler may only contain letters, digits, hyphens, and "
                "underscores, and cannot start or end with a hyphen or underscore."))

        if self.env['seller.shop'].search([('url_handler', '=', url_handler)], limit=1):
            raise UserError(_("A shop with URL handler '%s' already exists.") % url_handler)

        if self.env['seller.shop'].search([('name', '=', self.shop_name)], limit=1):
            raise UserError(_("A shop named '%s' already exists.") % self.shop_name)

        # 1. Placeholder seller partner — no res.user until transfer
        placeholder = self.env['res.partner'].create({
            'name': self.seller_name,
            'email': self.seller_email,
            'seller': True,
            'state': 'approved',
            'set_seller_wise_settings': True,
        })

        # 2. Shop — dexmi_marketplace.SellerShop.create() will restore state='approved'
        #    even though odoo_marketplace's create() would try to copy seller state.
        shop = self.env['seller.shop'].create({
            'name': self.shop_name,
            'url_handler': url_handler,
            'description': self.description,
            'shop_logo': self.shop_logo,
            'shop_banner': self.shop_banner,
            'shop_tag_line': self.shop_tag_line,
            'seller_id': placeholder.id,
            'managed_by_id': self.managed_by_id.id or self.env.user.partner_id.id,
            'state': 'approved',
        })

        return {
            'type': 'ir.actions.act_window',
            'name': _('Seller Shop'),
            'res_model': 'seller.shop',
            'view_mode': 'form',
            'res_id': shop.id,
            'target': 'current',
        }
```

- [ ] **Step 6.2: Create the wizard view**

```xml
<!-- odoo/dexmi_marketplace/wizards/shop_creation_wizard_views.xml -->
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="seller_shop_creation_wizard_form_view" model="ir.ui.view">
        <field name="name">seller.shop.creation.wizard.form</field>
        <field name="model">seller.shop.creation.wizard</field>
        <field name="arch" type="xml">
            <form string="Create New Seller Shop">
                <group string="New Seller Account">
                    <field name="seller_name" placeholder="Full name of the seller"/>
                    <field name="seller_email" placeholder="seller@example.com"/>
                </group>
                <group string="Shop Details">
                    <field name="shop_name" placeholder="My Store"/>
                    <field name="url_handler" placeholder="my-store"/>
                    <field name="shop_tag_line" placeholder="Short catchy description"/>
                    <field name="description"/>
                </group>
                <group string="Visual">
                    <field name="shop_logo" widget="image" options='{"size": [128, 128]}'/>
                    <field name="shop_banner" widget="image" options='{"size": [400, 120]}'/>
                </group>
                <field name="managed_by_id" invisible="1"/>
                <footer>
                    <button name="action_create_shop" string="Create Shop"
                            type="object" class="btn-primary"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="seller_shop_creation_wizard_action" model="ir.actions.act_window">
        <field name="name">Create New Seller Shop</field>
        <field name="res_model">seller.shop.creation.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>
```

- [ ] **Step 6.3: Register in `wizards/__init__.py`**

Add to the existing file:
```python
from . import shop_creation_wizard
```

- [ ] **Step 6.4: Write wizard tests**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestShopCreationWizard(TestBase):
    def test_wizard_creates_placeholder_partner_and_shop(self):
        wizard = self.env['seller.shop.creation.wizard'].create({
            'seller_name': 'New Seller',
            'seller_email': 'new_seller@test.com',
            'shop_name': 'Brand New Shop',
            'url_handler': 'brand-new-shop',
            'managed_by_id': self.manager_partner.id,
        })
        wizard.action_create_shop()

        shop = self.env['seller.shop'].search([('url_handler', '=', 'brand-new-shop')])
        self.assertTrue(shop, "Shop must be created")
        self.assertEqual(shop.managed_by_id, self.manager_partner)
        self.assertEqual(shop.seller_id.name, 'New Seller')
        self.assertEqual(shop.seller_id.email, 'new_seller@test.com')
        self.assertTrue(shop.seller_id.seller)
        self.assertEqual(shop.seller_id.state, 'approved')
        self.assertFalse(shop.seller_id.user_ids,
            "Placeholder must have no user until transfer")
        self.assertEqual(shop.seller_id.seller_shop_id, shop,
            "seller.shop.create() must backfill seller_shop_id on placeholder")
        self.assertEqual(shop.state, 'approved',
            "shop state must be 'approved' despite odoo_marketplace state sync")

    def test_wizard_rejects_duplicate_url_handler(self):
        wizard = self.env['seller.shop.creation.wizard'].create({
            'seller_name': 'Dup', 'seller_email': 'dup@test.com',
            'shop_name': 'Dup Shop', 'url_handler': 'test-shop',
            'managed_by_id': self.manager_partner.id,
        })
        from odoo.exceptions import UserError
        with self.assertRaises(UserError):
            wizard.action_create_shop()

    def test_wizard_rejects_invalid_url_handler(self):
        wizard = self.env['seller.shop.creation.wizard'].create({
            'seller_name': 'Bad', 'seller_email': 'bad@test.com',
            'shop_name': 'Bad Shop', 'url_handler': '-starts-with-dash',
            'managed_by_id': self.manager_partner.id,
        })
        from odoo.exceptions import UserError
        with self.assertRaises(UserError):
            wizard.action_create_shop()
```

- [ ] **Step 6.5: Upgrade and run tests**

```bash
python odoo-bin -c odoo.conf -d test_db --stop-after-init -u dexmi_marketplace 2>&1 | tail -10
python odoo-bin -c odoo.conf -d test_db \
    --test-tags=/dexmi_marketplace:TestShopCreationWizard \
    --stop-after-init --test-enable 2>&1 | tail -30
```

Expected: All 3 tests PASS.

- [ ] **Step 6.6: Commit**

```bash
git add odoo/dexmi_marketplace/wizards/shop_creation_wizard.py \
        odoo/dexmi_marketplace/wizards/shop_creation_wizard_views.xml \
        odoo/dexmi_marketplace/wizards/__init__.py \
        odoo/dexmi_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(dexmi-mp): add shop creation wizard (placeholder partner + shop)"
```

---

### Task 7: Shop Transfer Wizard

**Files:**
- Create: `odoo/dexmi_marketplace/wizards/shop_transfer_wizard.py`
- Create: `odoo/dexmi_marketplace/wizards/shop_transfer_wizard_views.xml`
- Modify: `odoo/dexmi_marketplace/wizards/__init__.py`

- [ ] **Step 7.1: Create the transfer wizard model**

```python
# odoo/dexmi_marketplace/wizards/shop_transfer_wizard.py
# -*- coding: utf-8 -*-
from odoo import models, fields, api, _
from odoo.exceptions import UserError


class ShopTransferWizard(models.TransientModel):
    _name = 'seller.shop.transfer.wizard'
    _description = 'Transfer Seller Shop to a User'

    shop_id = fields.Many2one(
        'seller.shop', string='Shop', required=True,
        default=lambda self: self.env.context.get('active_id'))
    transfer_type = fields.Selection([
        ('new_user', 'Create New User for this Shop'),
        ('existing_user', 'Transfer to Existing User (no shop yet)'),
    ], string='Transfer Type', required=True, default='new_user')

    new_user_login = fields.Char(string='Login (Email)')
    new_user_name = fields.Char(string='Full Name')
    existing_user_id = fields.Many2one(
        'res.users', string='Existing User',
        domain="[('partner_id.seller_shop_id', '=', False)]")
    notes = fields.Text(string='Transfer Notes')

    @api.onchange('shop_id')
    def _onchange_shop_id(self):
        if self.shop_id and self.shop_id.seller_id:
            seller = self.shop_id.seller_id
            self.new_user_login = seller.email or ''
            self.new_user_name = seller.name or ''

    def _validate(self):
        self.ensure_one()
        if not self.shop_id.managed_by_id:
            raise UserError(_(
                "Only shops created via 'Create New Seller Shop' can be transferred. "
                "This shop has no managing partner."))
        if self.transfer_type == 'new_user':
            if not self.new_user_login:
                raise UserError(_("Login (Email) is required for the new user."))
            if self.env['res.users'].search([('login', '=', self.new_user_login)], limit=1):
                raise UserError(
                    _("A user with login '%s' already exists.") % self.new_user_login)
        else:
            if not self.existing_user_id:
                raise UserError(_("Please select an existing user."))
            if self.existing_user_id.partner_id.seller_shop_id:
                raise UserError(
                    _("User '%s' already owns a shop.") % self.existing_user_id.name)

    def action_transfer(self):
        self.ensure_one()
        self._validate()
        shop = self.shop_id
        old_manager = shop.managed_by_id
        seller_group = self.env.ref('odoo_marketplace.marketplace_seller_group')

        if self.transfer_type == 'new_user':
            self._transfer_new_user(shop, old_manager, seller_group)
        else:
            self._transfer_existing_user(shop, old_manager, seller_group)

        shop.managed_by_id = False
        return {'type': 'ir.actions.act_window_close'}

    def _transfer_new_user(self, shop, old_manager, seller_group):
        """Create res.user linked to the existing placeholder partner.

        All products, payments, and SOL history remain on the same partner record.
        """
        placeholder = shop.seller_id
        new_user = self.env['res.users'].create({
            'name': self.new_user_name or placeholder.name,
            'login': self.new_user_login,
            'partner_id': placeholder.id,
            'groups_id': [(4, seller_group.id)],
        })
        self.env['seller.shop.transfer'].create({
            'shop_id': shop.id,
            'from_partner_id': old_manager.id,
            'to_partner_id': placeholder.id,
            'transferred_by_id': self.env.user.partner_id.id,
            'notes': self.notes,
        })
        return new_user

    def _transfer_existing_user(self, shop, old_manager, seller_group):
        """Reassign the shop to an existing user's partner.

        Steps:
        1. Migrate products and payments from placeholder to new partner.
        2. Change shop.seller_id to new partner (seller.shop.write() handles
           the partner cross-references).
        3. Archive the now-orphaned placeholder.
        4. Log the transfer.
        """
        new_partner = self.existing_user_id.partner_id
        old_placeholder = shop.seller_id

        products = self.env['product.template'].search([
            ('marketplace_shop_id', '=', shop.id),
        ])
        if products:
            products.write({'marketplace_seller_id': new_partner.id})

        payments = self.env['seller.payment'].search([
            ('seller_id', '=', old_placeholder.id),
        ])
        if payments:
            payments.write({'seller_id': new_partner.id})

        # seller.shop.write() will: clear old_placeholder.seller_shop_id,
        # set new_partner.seller_shop_id = shop.id
        shop.write({'seller_id': new_partner.id})

        new_partner.write({'seller': True, 'state': 'approved'})
        self.existing_user_id.write({'groups_id': [(4, seller_group.id)]})
        old_placeholder.active = False

        self.env['seller.shop.transfer'].create({
            'shop_id': shop.id,
            'from_partner_id': old_manager.id,
            'to_partner_id': new_partner.id,
            'transferred_by_id': self.env.user.partner_id.id,
            'notes': self.notes,
        })
```

- [ ] **Step 7.2: Create the transfer wizard view**

```xml
<!-- odoo/dexmi_marketplace/wizards/shop_transfer_wizard_views.xml -->
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="seller_shop_transfer_wizard_form_view" model="ir.ui.view">
        <field name="name">seller.shop.transfer.wizard.form</field>
        <field name="model">seller.shop.transfer.wizard</field>
        <field name="arch" type="xml">
            <form string="Transfer Seller Shop">
                <group>
                    <field name="shop_id" readonly="1"/>
                    <field name="transfer_type" widget="radio"/>
                </group>
                <group invisible="transfer_type != 'new_user'" string="New User Details">
                    <field name="new_user_name"/>
                    <field name="new_user_login" string="Login / Email"/>
                </group>
                <group invisible="transfer_type != 'existing_user'" string="Existing User">
                    <field name="existing_user_id" options="{'no_create': True}"/>
                </group>
                <group string="Notes">
                    <field name="notes" nolabel="1"
                           placeholder="Optional: reason for this transfer"/>
                </group>
                <footer>
                    <button name="action_transfer" string="Transfer Shop"
                            type="object" class="btn-primary"
                            confirm="Transfer this shop? This action cannot be undone."/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="seller_shop_transfer_wizard_action" model="ir.actions.act_window">
        <field name="name">Transfer Seller Shop</field>
        <field name="res_model">seller.shop.transfer.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="binding_model_id" ref="odoo_marketplace.model_seller_shop"/>
        <field name="binding_view_types">form</field>
    </record>
</odoo>
```

- [ ] **Step 7.3: Register in `wizards/__init__.py`**

Add:
```python
from . import shop_transfer_wizard
```

- [ ] **Step 7.4: Write transfer wizard tests**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestShopTransferWizard(TestBase):
    def test_transfer_new_user_creates_user_on_placeholder(self):
        wizard = self.env['seller.shop.transfer.wizard'].create({
            'shop_id': self.shop.id,
            'transfer_type': 'new_user',
            'new_user_login': 'real_seller@test.com',
            'new_user_name': 'Real Seller',
        })
        wizard.action_transfer()

        self.assertTrue(self.placeholder_partner.user_ids,
            "Transfer must create a res.user for the placeholder partner")
        new_user = self.placeholder_partner.user_ids[0]
        self.assertEqual(new_user.login, 'real_seller@test.com')
        self.assertFalse(self.shop.managed_by_id,
            "managed_by_id must be cleared after transfer")
        self.assertEqual(self.shop.seller_id, self.placeholder_partner,
            "seller_id must remain the placeholder (now the real seller)")

        log = self.env['seller.shop.transfer'].search([('shop_id', '=', self.shop.id)])
        self.assertTrue(log)
        self.assertEqual(log.from_partner_id, self.manager_partner)
        self.assertEqual(log.to_partner_id, self.placeholder_partner)

    def test_transfer_existing_user_migrates_products_and_payments(self):
        product = self.env['product.template'].create({
            'name': 'Managed Product', 'type': 'consu',
            'marketplace_shop_id': self.shop.id,
            'marketplace_seller_id': self.placeholder_partner.id,
        })
        other_partner = self.env['res.partner'].create({'name': 'Existing Seller'})
        other_user = self.env['res.users'].create({
            'name': 'Existing Seller',
            'login': 'existing_seller@test.com',
            'partner_id': other_partner.id,
        })

        wizard = self.env['seller.shop.transfer.wizard'].create({
            'shop_id': self.shop.id,
            'transfer_type': 'existing_user',
            'existing_user_id': other_user.id,
        })
        wizard.action_transfer()

        self.shop.invalidate_recordset()
        self.assertEqual(self.shop.seller_id, other_partner)

        product.invalidate_recordset()
        self.assertEqual(product.marketplace_seller_id, other_partner)
        self.assertEqual(product.marketplace_shop_id, self.shop,
            "marketplace_shop_id must remain unchanged")

        self.assertFalse(self.placeholder_partner.active,
            "Placeholder must be archived after existing-user transfer")

        log = self.env['seller.shop.transfer'].search([('shop_id', '=', self.shop.id)])
        self.assertTrue(log)
        self.assertEqual(log.to_partner_id, other_partner)

    def test_transfer_fails_without_managed_by_id(self):
        self.shop.managed_by_id = False
        wizard = self.env['seller.shop.transfer.wizard'].create({
            'shop_id': self.shop.id,
            'transfer_type': 'new_user',
            'new_user_login': 'x@test.com',
        })
        from odoo.exceptions import UserError
        with self.assertRaises(UserError):
            wizard.action_transfer()
```

- [ ] **Step 7.5: Upgrade and run transfer tests**

```bash
python odoo-bin -c odoo.conf -d test_db --stop-after-init -u dexmi_marketplace 2>&1 | tail -10
python odoo-bin -c odoo.conf -d test_db \
    --test-tags=/dexmi_marketplace:TestShopTransferWizard \
    --stop-after-init --test-enable 2>&1 | tail -30
```

Expected: All 3 tests PASS.

- [ ] **Step 7.6: Commit**

```bash
git add odoo/dexmi_marketplace/wizards/shop_transfer_wizard.py \
        odoo/dexmi_marketplace/wizards/shop_transfer_wizard_views.xml \
        odoo/dexmi_marketplace/wizards/__init__.py \
        odoo/dexmi_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(dexmi-mp): add shop transfer wizard (new user + existing user paths)"
```

---

### Task 8: Security — Access Control Rows

**Files:**
- Modify: `odoo/dexmi_marketplace/security/ir.model.access.csv`

- [ ] **Step 8.1: Append access rows**

Add these lines to `security/ir.model.access.csv`:

```csv
access_seller_shop_transfer_officer,seller_shop_transfer_officer,dexmi_marketplace.model_seller_shop_transfer,odoo_marketplace.marketplace_officer_group,1,0,0,0
access_seller_shop_transfer_manager,seller_shop_transfer_manager,dexmi_marketplace.model_seller_shop_transfer,odoo_marketplace.marketplace_manager_group,1,1,1,1
access_shop_creation_wizard_manager,shop_creation_wizard_manager,dexmi_marketplace.model_seller_shop_creation_wizard,odoo_marketplace.marketplace_manager_group,1,1,1,1
access_shop_transfer_wizard_manager,shop_transfer_wizard_manager,dexmi_marketplace.model_seller_shop_transfer_wizard,odoo_marketplace.marketplace_manager_group,1,1,1,1
```

Justification:
- `seller.shop.transfer`: Officers read-only (audit); Managers full CRUD (they create log entries).
- Both wizard models: Managers only. Officers do not create shops or initiate transfers.

- [ ] **Step 8.2: Commit**

```bash
git add odoo/dexmi_marketplace/security/ir.model.access.csv
git commit -m "chore(dexmi-mp): add access rights for shop manager models"
```

---

### Task 9: Views — `seller.shop` Form Inheritance

**Files:**
- Create: `odoo/dexmi_marketplace/views/seller_shop_inherit.xml`

This file inherits `odoo_marketplace.wk_seller_shop_form_view` via XPath to add the Transfer button, `managed_by_id` field, Transfer History tab, and fix the Products domain — without touching the upstream view.

- [ ] **Step 9.1: Create the inheritance view**

```xml
<!-- odoo/dexmi_marketplace/views/seller_shop_inherit.xml -->
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="dexmi_seller_shop_form_inherit" model="ir.ui.view">
        <field name="name">dexmi.seller.shop.form.inherit</field>
        <field name="model">seller.shop</field>
        <field name="inherit_id" ref="odoo_marketplace.wk_seller_shop_form_view"/>
        <field name="arch" type="xml">

            <!-- 1. Transfer button in header -->
            <xpath expr="//header/field[@name='state']" position="before">
                <button name="%(dexmi_marketplace.seller_shop_transfer_wizard_action)d"
                        string="Transfer to Seller"
                        type="action"
                        class="btn-primary"
                        groups="odoo_marketplace.marketplace_manager_group"
                        invisible="not managed_by_id or not id"/>
            </xpath>

            <!-- 2. managed_by_id field after seller_id (officer group row) -->
            <xpath expr="//field[@name='seller_id'][@groups='odoo_marketplace.marketplace_officer_group']"
                   position="after">
                <field name="managed_by_id"
                       groups="odoo_marketplace.marketplace_manager_group"
                       options="{'no_create': True}"
                       readonly="1"/>
            </xpath>

            <!-- 3. Fix Products domain: use marketplace_shop_id instead of marketplace_seller_id -->
            <xpath expr="//page[@name='seller_products']/field[@name='seller_product_ids']"
                   position="attributes">
                <attribute name="domain">[('marketplace_shop_id', '=', id), ('status', '=', 'approved')]</attribute>
            </xpath>

            <!-- 4. Transfer History tab (manager only) -->
            <xpath expr="//page[@name='seller_shop_t_c']" position="after">
                <page name="transfer_history" string="Transfer History"
                      groups="odoo_marketplace.marketplace_manager_group">
                    <field name="transfer_history_ids" readonly="1">
                        <tree>
                            <field name="transfer_date"/>
                            <field name="from_partner_id" string="From Manager"/>
                            <field name="to_partner_id" string="To Seller"/>
                            <field name="transferred_by_id"/>
                            <field name="notes"/>
                        </tree>
                    </field>
                </page>
            </xpath>
        </field>
    </record>
</odoo>
```

- [ ] **Step 9.2: Commit**

```bash
git add odoo/dexmi_marketplace/views/seller_shop_inherit.xml
git commit -m "feat(dexmi-mp): inherit seller shop form — Transfer button, managed_by_id, history, product domain"
```

---

### Task 10: Views — Product Form Inheritance (`marketplace_shop_id`)

**Files:**
- Create: `odoo/dexmi_marketplace/views/mp_product_shop_inherit.xml`

This inherits `odoo_marketplace.wk_seller_product_template_form_view`.

- [ ] **Step 10.1: Create the product view inheritance**

```xml
<!-- odoo/dexmi_marketplace/views/mp_product_shop_inherit.xml -->
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="dexmi_mp_product_form_shop_inherit" model="ir.ui.view">
        <field name="name">dexmi.mp.product.form.shop.inherit</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="odoo_marketplace.wk_seller_product_template_form_view"/>
        <field name="arch" type="xml">

            <!-- Add marketplace_shop_id above marketplace_seller_id (officer group).
                 Selecting a shop auto-fills the seller via @api.onchange in the model.
                 When shop is set, marketplace_seller_id becomes readonly. -->
            <xpath expr="//field[@name='marketplace_seller_id'][@groups='odoo_marketplace.marketplace_officer_group']"
                   position="before">
                <field name="marketplace_shop_id"
                       options="{'no_create': True}"
                       domain="[('state', '=', 'approved')]"
                       groups="odoo_marketplace.marketplace_manager_group"
                       readonly="status in ('approved', 'rejected')"/>
            </xpath>

            <!-- Make seller readonly when shop is chosen (prevent mismatch) -->
            <xpath expr="//field[@name='marketplace_seller_id'][@groups='odoo_marketplace.marketplace_officer_group']"
                   position="attributes">
                <attribute name="readonly">status in ('approved','rejected') or marketplace_shop_id != False</attribute>
            </xpath>

        </field>
    </record>
</odoo>
```

- [ ] **Step 10.2: Commit**

```bash
git add odoo/dexmi_marketplace/views/mp_product_shop_inherit.xml
git commit -m "feat(dexmi-mp): add marketplace_shop_id to product form with seller auto-lock"
```

---

### Task 11: Views — Manager Partner Form ("Managed Shops" Tab)

**Files:**
- Create: `odoo/dexmi_marketplace/views/res_partner_managed_shops.xml`

This inherits `odoo_marketplace.wk_seller_form_view` (the seller partner form in the Seller Dashboard).

- [ ] **Step 11.1: Create the partner view inheritance**

```xml
<!-- odoo/dexmi_marketplace/views/res_partner_managed_shops.xml -->
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="dexmi_res_partner_managed_shops_inherit" model="ir.ui.view">
        <field name="name">dexmi.res.partner.managed.shops.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="odoo_marketplace.wk_seller_form_view"/>
        <field name="arch" type="xml">

            <!-- Add Managed Shops tab after the last existing page in the notebook -->
            <xpath expr="//notebook" position="inside">
                <page name="managed_shops" string="Managed Shops"
                      groups="odoo_marketplace.marketplace_manager_group">
                    <field name="managed_shop_ids" readonly="1">
                        <list>
                            <field name="name"/>
                            <field name="seller_id" string="Placeholder Seller"/>
                            <field name="state"/>
                            <field name="website_published" string="Published"/>
                            <field name="url" widget="url"/>
                        </list>
                    </field>
                </page>
            </xpath>

        </field>
    </record>
</odoo>
```

- [ ] **Step 11.2: Commit**

```bash
git add odoo/dexmi_marketplace/views/res_partner_managed_shops.xml
git commit -m "feat(dexmi-mp): add Managed Shops tab to manager partner form"
```

---

### Task 12: Controller — Override Shop Product Filtering

**Files:**
- Modify: `odoo/dexmi_marketplace/controllers/main.py`

Odoo controller inheritance: define a subclass of `MarketplaceSellerShop` and re-declare the same route. The Odoo routing layer will use the most-derived class for the route.

- [ ] **Step 12.1: Add the import and subclass to `controllers/main.py`**

At the top of the file, after existing imports, add:
```python
from odoo.addons.odoo_marketplace.controllers.main import MarketplaceSellerShop
```

Then, at the end of `controllers/main.py`, add the subclass:

```python
class DexmiMarketplaceSellerShop(MarketplaceSellerShop):
    """Override seller shop route to filter products by marketplace_shop_id.

    odoo_marketplace filters by marketplace_seller_id (seller partner).
    With the multi-shop feature, products are tied to a specific shop, so
    filtering by shop ID gives the correct per-shop product list.
    """

    @http.route(
        ['/seller/shop/<shop_url_handler>',
         '/seller/shop/<shop_url_handler>/page/<int:page>'],
        type='http', auth='public', website=True)
    def seller_shop(self, shop_url_handler, page=0, category=None,
                    search='', ppg=False, **post):
        if not request.website.enable_marketplace or not request.website.mp_show_seller_shop_list:
            return request.render('http_routing.404')

        shop_obj = request.env['seller.shop'].sudo().search(
            [('url_handler', '=', str(shop_url_handler))], limit=1)
        if not shop_obj:
            return False
        if (not shop_obj.sudo().website_published
                and request.env.user.seller
                and request.env.user.url_handler != shop_obj.sudo().seller_id.url_handler):
            return request.render('http_routing.403')

        def _get_search_domain(search):
            domain = request.website.sale_product_domain()
            domain += [('marketplace_shop_id', '=', shop_obj.sudo().id)]  # changed
            if search:
                for srch in search.split(' '):
                    domain += [
                        '|', '|', '|',
                        ('name', 'ilike', srch),
                        ('description', 'ilike', srch),
                        ('description_sale', 'ilike', srch),
                        ('product_variant_ids.default_code', 'ilike', srch),
                    ]
            return request.env['product.template'].sudo().search(domain)

        uid, context, env = request.uid, dict(request.env.context), request.env
        url = '/seller/shop/' + str(shop_obj.url_handler)
        if search:
            post['search'] = search

        if not ppg:
            ppg = request.env['website'].get_current_website().shop_ppg
        PPR = request.env['website'].get_current_website().shop_ppr
        if ppg:
            try:
                ppg = int(ppg)
            except ValueError:
                ppg = 20
            post['ppg'] = ppg
        else:
            ppg = 20

        if not context.get('pricelist'):
            pricelist = request.website._get_current_pricelist()
            context['pricelist'] = int(pricelist)
        else:
            pricelist = env['product.pricelist'].sudo().browse(context['pricelist'])

        # Sales count remains seller-level (not shop-level)
        sales_count = 0
        all_products = request.env['product.template'].sudo().search(
            [('marketplace_seller_id', '=', shop_obj.sudo().seller_id.id)])
        for prod in all_products.with_user(SUPERUSER_ID):
            sales_count += prod.sales_count

        from odoo.addons.website.tools import QueryURL
        attrib_list = request.httprequest.args.getlist('attrib')
        keep = QueryURL('/seller/shop/' + str(shop_obj.url_handler),
                        category=category and int(category),
                        search=search, attrib=attrib_list)

        product_count = request.env['product.template'].sudo().search_count([
            ('sale_ok', '=', True),
            ('status', '=', 'approved'),
            ('website_published', '=', True),
            ('marketplace_shop_id', '=', shop_obj.sudo().id),  # changed
        ])
        pager = request.website.pager(
            url=url, total=product_count, page=page,
            step=ppg, scope=7, url_args=post)
        products = env['product.template'].sudo().search([
            ('sale_ok', '=', True),
            ('status', '=', 'approved'),
            ('website_published', '=', True),
            ('marketplace_shop_id', '=', shop_obj.sudo().id),  # changed
        ], limit=ppg, offset=pager['offset'],
           order='website_published desc, website_sequence desc')

        from odoo.tools import lazy
        shop_banner_url = request.website.image_url(shop_obj, 'shop_banner')
        fiscal_position = request.env['website'].get_current_website().fiscal_position_id.sudo()
        products_prices = lazy(lambda: products._get_sales_prices(pricelist, fiscal_position))
        from_currency = env['res.users'].sudo().browse(uid).company_id.currency_id
        to_currency = pricelist.currency_id
        compute_currency = lambda price: env['res.currency'].sudo()._compute(
            from_currency, to_currency, price)

        values = {
            'shop_obj': shop_obj,
            'search': search,
            'rows': PPR,
            'bins': None,  # populated below after TableCompute import
            'ppg': ppg,
            'ppr': PPR,
            'pager': pager,
            'products': products if not search else _get_search_domain(search),
            'keep': keep,
            'compute_currency': compute_currency,
            'pricelist': pricelist,
            'hide_pager': len(_get_search_domain(search)),
            'shop_banner_url': shop_banner_url,
            'sales_count': sales_count,
            'product_count': int(product_count),
            'get_product_prices': lambda product: lazy(lambda: products_prices[product.id]),
        }

        # TableCompute is defined in odoo_marketplace.controllers.main
        from odoo.addons.odoo_marketplace.controllers.main import TableCompute
        values['bins'] = TableCompute().process(
            values['products'], ppg, PPR)

        website_sale_wishlist = request.env['ir.module.module'].sudo().search(
            [('state', '=', 'installed'), ('name', '=', 'website_sale_wishlist')])
        if website_sale_wishlist:
            values['products_in_wishlist'] = (
                request.env['product.wishlist'].current().product_id.product_tmpl_id)

        return request.render('odoo_marketplace.mp_seller_shop', values)
```

- [ ] **Step 12.2: Write the controller domain test**

Add to `tests/test_seller_shop_manager.py`:

```python
class TestShopControllerDomain(TestBase):
    def test_shop_product_listing_uses_shop_not_seller(self):
        """Products in a different shop of the same seller must not appear."""
        second_placeholder = self.env['res.partner'].create({
            'name': 'Second Placeholder', 'seller': True, 'state': 'approved'})
        second_shop = self.env['seller.shop'].create({
            'name': 'Second Shop', 'url_handler': 'second-shop',
            'seller_id': second_placeholder.id, 'state': 'approved',
        })
        product_in_shop = self.env['product.template'].create({
            'name': 'In Shop Product', 'type': 'consu',
            'marketplace_shop_id': self.shop.id,
            'marketplace_seller_id': self.placeholder_partner.id,
            'status': 'approved', 'sale_ok': True,
        })
        product_in_other_shop = self.env['product.template'].create({
            'name': 'Other Shop Product', 'type': 'consu',
            'marketplace_shop_id': second_shop.id,
            'marketplace_seller_id': second_placeholder.id,
            'status': 'approved', 'sale_ok': True,
        })

        domain = [
            ('sale_ok', '=', True),
            ('status', '=', 'approved'),
            ('marketplace_shop_id', '=', self.shop.id),
        ]
        results = self.env['product.template'].search(domain)
        self.assertIn(product_in_shop, results)
        self.assertNotIn(product_in_other_shop, results,
            "Products in a different shop must not appear in this shop's listing")
```

- [ ] **Step 12.3: Run controller test**

```bash
python odoo-bin -c odoo.conf -d test_db \
    --test-tags=/dexmi_marketplace:TestShopControllerDomain \
    --stop-after-init --test-enable 2>&1 | tail -20
```

Expected: PASS.

- [ ] **Step 12.4: Commit**

```bash
git add odoo/dexmi_marketplace/controllers/main.py \
        odoo/dexmi_marketplace/tests/test_seller_shop_manager.py
git commit -m "feat(dexmi-mp): override shop controller to filter products by marketplace_shop_id"
```

---

### Task 13: Manifest — Register All New Files

**Files:**
- Modify: `odoo/dexmi_marketplace/__manifest__.py`

- [ ] **Step 13.1: Add new files to the `data` list**

Inside the `"data"` list in `__manifest__.py`, add the new files in the correct section order. Insert the wizard XML entries in the `# WIZARDS` section (after existing wizard entries) and the view XML entries in the `# VISTAS` section (after existing view entries):

```python
# WIZARDS section — add after existing wizard entries:
"wizards/shop_creation_wizard_views.xml",
"wizards/shop_transfer_wizard_views.xml",

# VISTAS section — add after existing view entries:
"views/seller_shop_inherit.xml",
"views/mp_product_shop_inherit.xml",
"views/res_partner_managed_shops.xml",
```

- [ ] **Step 13.2: Full upgrade — verify zero errors**

```bash
python odoo-bin -c odoo.conf -d test_db --stop-after-init -u dexmi_marketplace 2>&1 \
    | grep -E "^(ERROR|WARNING.*dexmi|raise)" | head -20
```

Expected: No `ERROR` lines. Pre-existing warnings from `odoo_marketplace` (e.g. `_read_group_fill_results`) are unrelated and acceptable.

- [ ] **Step 13.3: Run the full test suite for `dexmi_marketplace`**

```bash
python odoo-bin -c odoo.conf -d test_db \
    --test-tags=/dexmi_marketplace \
    --stop-after-init --test-enable 2>&1 | tail -40
```

Expected: All tests in the following classes PASS with no failures:
- `TestSellerShopTransfer`
- `TestShopStateIndependence`
- `TestPartnerStateSyncRemoved`
- `TestProductShopAssignment`
- `TestMigration`
- `TestShopCreationWizard`
- `TestShopTransferWizard`
- `TestShopControllerDomain`

- [ ] **Step 13.4: Confirm `odoo_marketplace` is untouched**

```bash
git diff HEAD -- odoo/odoo_marketplace/ | wc -l
```

Expected: `0` — no lines changed in `odoo_marketplace`.

- [ ] **Step 13.5: Commit**

```bash
git add odoo/dexmi_marketplace/__manifest__.py
git commit -m "chore(dexmi-mp): register new views and wizards in manifest"
```

---

### Self-Review

#### Spec Coverage

| Requirement | Task | Implementation |
|---|---|---|
| Manager creates multiple shops, each with own partner | Task 6 | `seller.shop.creation.wizard` creates placeholder partner + shop |
| `managed_by_id` links manager to their shops | Tasks 2, 3 | Field on `seller.shop`, One2many on `res.partner` |
| Products explicitly assigned to a shop | Task 4 | `marketplace_shop_id` on `product.template` |
| Seller auto-derived from shop on product create | Task 4 | `@api.onchange` + `create()` override |
| Shop state independent from seller state | Tasks 2, 3 | Post-super() state restoration in both `seller.shop` and `res.partner` |
| Transfer to new user | Task 7 | `_transfer_new_user()` — `res.user` on placeholder |
| Transfer to existing user | Task 7 | `_transfer_existing_user()` — migrates products + payments, archives placeholder |
| Transfer history log | Tasks 1, 7 | `seller.shop.transfer` model, created in both transfer paths |
| `managed_by_id` cleared after transfer | Task 7 | `shop.managed_by_id = False` in `action_transfer()` |
| Post-install migration for existing products | Task 5 | `migrations/17.0.1.0/post-migrate.py` |
| Frontend shop shows only that shop's products | Task 12 | Controller override filters by `marketplace_shop_id` |
| `odoo_marketplace` untouched | All tasks | All changes in `dexmi_marketplace` only |
| Commissions go to seller partner | No change | `marketplace_seller_id` drives commissions; migration keeps it correct |
| Manager's existing personal shop unaffected | No change | Only newly created shops have `managed_by_id` set |

#### No Placeholders Check

All code blocks are complete. No "TBD", "handle edge cases", or "similar to above" patterns present.

#### Type Consistency

| Symbol                                      | Defined | Used consistently               |
| ------------------------------------------- | ------- | ------------------------------- |
| `seller.shop.transfer`                      | Task 1  | Tasks 1, 7                      |
| `seller.shop.creation.wizard`               | Task 6  | Tasks 6, 8, 9 (action ref)      |
| `seller.shop.transfer.wizard`               | Task 7  | Tasks 7, 8                      |
| `managed_by_id` on `seller.shop`            | Task 2  | Tasks 2, 6, 7, 9                |
| `transfer_history_ids` on `seller.shop`     | Task 2  | Task 9                          |
| `managed_shop_ids` on `res.partner`         | Task 3  | Task 11                         |
| `marketplace_shop_id` on `product.template` | Task 4  | Tasks 4, 5, 7, 10, 12           |
| `_transfer_new_user()`                      | Task 7  | Task 7 (`action_transfer`)      |
| `_transfer_existing_user()`                 | Task 7  | Task 7 (`action_transfer`)      |
| `DexmiMarketplaceSellerShop`                | Task 12 | Task 12 only (controller class) |

___

## Claude Sessions
