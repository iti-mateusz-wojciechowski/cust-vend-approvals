# Customer & Vendor Unblock Approvals

**Two ready-to-import approval workflows that hold back any loosening of the Blocked field on a
Customer or Vendor until an approver signs it off** in Microsoft Dynamics 365 Business Central.

Companion sample for the article
[Guilty until approved - Customer & Vendor Approvals in Business Central](https://bc-workshop.com/blog/customer-and-vendor-approvals-in-business-central).

| | |
|---|---|
| Format | Business Central workflow export (XML), imported from the **Workflows** page |
| Exported from | Business Central, February 2026 |
| Workflow codes | `BCW-UCAPW-01` (customer), `BCW-UVAPW-01` (vendor) |
| Dependencies | none (standard workflow events and responses only, no extension) |
| Licence | MIT |

## The problem

Standard Business Central ships a **Customer Approval Workflow** and a **Vendor Approval Workflow**,
but both start only when someone chooses **Send Approval Request** on the card. Nothing forces that
step, so a new customer can be created and invoiced without anyone approving it.

The **Blocked** field is the control that actually stops posting. Make every new customer or vendor
start blocked, and the question becomes *who may unblock it* — which is what these workflows govern.

## What these workflows add

| Feature | Detail |
|---|---|
| Approval on any loosening of **Blocked** | Lowering **Blocked** on a **Customer** (`All` → `Invoice` → `Ship` → blank) or a **Vendor** (`All` → `Payment` → blank) sends an approval request instead of taking effect. |
| Old value stays until approved | The record keeps its previous **Blocked** value, and its use is restricted while the request is pending. |
| Tightening is never held up | Raising **Blocked** takes effect immediately, with no approval. |
| Approvals go to a workflow user group | Customer requests go to the workflow user group `CUSTOMER`, vendor requests to `VENDOR`. |

## How it works

Both workflows are built on the standard **Customer Credit Limit Change Approval Workflow** pattern,
with the trigger rule moved from **Credit Limit (LCY)** to **Blocked** (field 39 on **Customer**,
table 18, and on **Vendor**, table 23) and its operator set to *Decreased*.

| When | The workflow |
|---|---|
| A user lowers **Blocked** | Restricts the record's usage, reverts **Blocked** to the previous value, creates and sends approval requests to the workflow user group, and tells the user: *"A request to decrease the customer locking level has been sent for approval."* (vendor wording for vendors) |
| The last pending request is approved | Lifts the restriction and applies the new **Blocked** value |
| A request is approved and more are pending | Sends the next request in the group's sequence |
| A request is rejected | Lifts the restriction, discards the new value, rejects all open requests, and notifies the sender |
| A request is delegated | Sends it to the substitute approver |

## Installing

1. In **Approval User Setup**, add every user who will change **Blocked** or approve the change.
   Without an entry, sending the request fails for that user.
2. In **Workflow User Groups**, create the groups `CUSTOMER` and `VENDOR` and add the approvers with
   their sequence numbers. Without them, the workflows have no one to send the request to.
3. Open **Workflows**, choose **Import from File**, and import `UnblockCustomer.xml`. Repeat for
   `UnblockVendor.xml`.
4. Open each imported workflow and turn on **Enabled**. Imported workflows arrive disabled.
5. Set **Blocked** to `All` on your **Customer Templates** and **Vendor Templates**, so new records
   created from a template start blocked. Skip this, and new records start unblocked and never reach
   the workflow.

## Limitations

- **Unblocking only.** These files do not block new records. Template defaults cover only records
  created from a template; a customer or vendor created any other way starts unblocked. The article
  recommends a small extension that forces **Blocked** to `All` on insert — it is not part of this
  repository.
- **Fixed group codes.** The approver groups are `CUSTOMER` and `VENDOR`. To use other groups, change
  the **Create an approval request** response after import.
- **Built-in workflows only.** The Power Automate variant described in the article, using the
  `CustomerBlocked` and `CustomerUnBlocked` business events, is not included.

## File map

| Path | Workflow | Purpose |
|---|---|---|
| `UnblockCustomer.xml` | `BCW-UCAPW-01` Unblock Customer Approval Workflow (category `SALES`) | Approval on lowering **Blocked** on a **Customer** |
| `UnblockVendor.xml` | `BCW-UVAPW-01` Unblock Vendor Approval Workflow (category `PURCH`) | Approval on lowering **Blocked** on a **Vendor** |

## Licence

MIT — see [LICENSE](LICENSE).
