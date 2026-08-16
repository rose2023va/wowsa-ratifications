# Airtable

Base: **Ratifications Pipeline**, `appgUjmgd0K8WWp31`

---

## Tables

| Tab | Table ID | Role |
|---|---|---|
| Workflow | `tblulYhWIpP0E3eov` | The tracker. Updated manually at every stage. Nothing writes to it. |
| Table 1 | `tblAPf6E4uzRS0oCE` | Where orders land from the automation. |
| Pre-Swim Planning | To confirm | Contents not yet documented. See [13-open-questions.md](13-open-questions.md). |

---

## Table 1 fields

Eight records at time of capture.

| Field | Type | Populated by |
|---|---|---|
| SWIM ID# | Formula | Derived |
| SWIM ID | Text, currently hidden | Jotform |
| Email Address | Email | Order automation, from billing |
| Order ID | Text | Order automation |
| Order Date | Date | Order automation |
| Amount Paid | Currency | Order automation |
| First Name | Text | Order automation, from billing |
| Last Name | Text | Order automation, from billing |
| Swim Date | Date | To confirm |
| Swimmer's First Name | Text | To confirm |
| Swimmer's Last Name | Text | To confirm |
| Swimmer's Email Address | Email | To confirm |
| Auto ID | Autonumber | To confirm |
| Product | Text | **Nothing** |
| Add-ons | Long text | **Nothing** |
| Record Attempt | Checkbox | **Nothing** |
| Live Map Tracking | Checkbox | **Nothing** |
| Blog Recap & Social Sharing | Checkbox | **Nothing** |
| Order Link | URL | **Nothing** |

The buyer and the swimmer are held separately. First Name and Last Name come from billing;
the Swimmer's fields are distinct, so the person paying is not assumed to be the person swimming.

The six fields marked **Nothing** were added in preparation for capturing product variation and
add-ons. The script that would populate them was drafted and never applied.

---

## The Ratification Order automation

| Item | Value |
|---|---|
| Automation ID | `wac1dXCCGGJzzisj8` |
| Builder | https://airtable.com/appgUjmgd0K8WWp31/wflxdCC6f3GFBBwHq/wac1dXCCGGJzzisj8 |
| State | On |
| Runs this month | 5 |
| Last updated by | Quinn Fitzgerald |

### Shape

| Step | Type | What it does |
|---|---|---|
| Trigger | When webhook received | Receives the WooCommerce order payload |
| Action 1 | Run a script | Filters for the ratification product and extracts order fields |
| Action 2 | Conditional, `If matchFound is true` | Create record in Table 1, then Gmail: Send email |

Orders that are not the ratification product fall through and nothing is created.

### Field mapping on Create record

| Airtable field | Script output |
|---|---|
| Order ID | `orderId` |
| Email Address | `customerEmail` |
| Order Date | `orderDate` |
| Amount Paid | `amountPaid` |
| First Name | `firstName` |
| Last Name | `lastName` |

### The live script

```javascript
let inputConfig = input.config();
let payload = inputConfig.payload;
let order = payload.body ? payload.body : payload;

let RATIFICATION_PRODUCT_ID = 70664;
let matchFound = false;
let lineItems = order.line_items || [];

for (let item of lineItems) {
    if (item.product_id === RATIFICATION_PRODUCT_ID) {
        matchFound = true;
        break;
    }
}

if (!matchFound) {
    output.set('matchFound', false);
} else {
    let orderId = order.id;
    let customerEmail = order.billing ? order.billing.email : '';
    let firstName = order.billing ? order.billing.first_name : '';
    let lastName = order.billing ? order.billing.last_name : '';
    let orderDate = order.date_created;
    let amountPaid = order.total;

    output.set('matchFound', true);
    output.set('orderId', String(orderId));
    output.set('customerEmail', customerEmail);
    output.set('firstName', firstName);
    output.set('lastName', lastName);
    output.set('orderDate', orderDate);
    output.set('amountPaid', amountPaid);
}
```

### Current fault state

- The Run a script step shows **Fix configuration**
- The `payload` input reads **Invalid value**
- **Trigger test failed**, shown twice

Two candidate causes, neither confirmed:

1. An unfinished attempt to add Solo and Relay variation handling. The live script above contains
   no variation logic, so if that work was started it was not completed.
2. The trigger has never had a sample webhook payload captured, so the script step has no payload
   to validate against. This condition was present before the variation work began.

### What the script does not do

**It never reads the variation.** Both variation IDs, Solo `102119` and Relay `102120`, arrive on
the line item and are discarded. The script matches on `product_id` alone.

**It never reads add-ons.** Nothing populates Product, Add-ons, the three add-on checkboxes, or
Order Link.

### Why the fix has not been applied

Airtable's API refuses to edit an automation containing a script step, rejecting it as a
read-only node. The change must be pasted in through the Airtable interface by a person.

An extended version covering product, add-ons and order link was drafted on 6 August 2026,
was never applied, and was not saved anywhere. It is gone.

---

## The Workflow table

This is the tracker a swimmer's status is checked against when they ask where their ratification
stands.

**Nothing writes to it.** The order automation writes to Table 1. Jotform does not write to it.
The Google Drive nodes do not write to it. Every status change, every stage advance and every
note is typed in by a person after doing the underlying work in another system.

It is therefore a parallel account of a process happening elsewhere, accurate only to the extent
that someone remembered to update it.
