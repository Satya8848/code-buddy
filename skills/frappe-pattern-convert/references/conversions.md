# Conversion Recipes

## Raw SQL → frappe.qb
```python
# before
frappe.db.sql(f"""select name, grand_total from `tabSales Invoice`
  where customer = '{customer}' and docstatus = 1 order by posting_date desc""", as_dict=1)

# after
SI = frappe.qb.DocType("Sales Invoice")
(
    frappe.qb.from_(SI)
    .select(SI.name, SI.grand_total)
    .where((SI.customer == customer) & (SI.docstatus == 1))
    .orderby(SI.posting_date, order=frappe.qb.desc)
).run(as_dict=True)
```
Simple single-table reads → `frappe.get_all("Sales Invoice", filters={...}, fields=[...], order_by="posting_date desc")`.

Aggregates: `from frappe.query_builder.functions import Sum, Count` → `.select(SI.customer, Sum(SI.grand_total).as_("total")).groupby(SI.customer)`.

Joins: `.from_(SI).inner_join(SII).on(SII.parent == SI.name)`.

## get_doc loop → batched read
```python
# before
for name in names:
    doc = frappe.get_doc("Item", name)
    rates[name] = doc.standard_rate
# after
rates = {
    d.name: d.standard_rate
    for d in frappe.get_all("Item", filters={"name": ["in", names]}, fields=["name", "standard_rate"])
}
```

## get_value in loop → single dict
`frappe.get_all(..., fields=["name","x"])` then dict lookup; or `frappe.db.get_values(doctype, {"name": ["in", names]}, ["name","x"], as_dict=True)`.

## Repeated settings reads
`frappe.get_doc("Selling Settings")` → `frappe.get_cached_doc("Selling Settings")` / `frappe.db.get_single_value("Selling Settings", "field")`.

## Commit removal
Remove `frappe.db.commit()` from request-cycle code. If the code needs durability for a long operation, move it into an enqueued job that commits per batch.

## Sync heavy work → enqueue
`do_heavy(doc)` in `on_submit` → `frappe.enqueue("path.do_heavy", queue="long", job_id=f"heavy-{doc.name}", deduplicate=True, enqueue_after_commit=True, name=doc.name)`.

## JS callbacks → async/await
```js
// before
frappe.call({ method: "my_app.api.x", args: {a}, callback(r) { frm.set_value("b", r.message); } });
// after
const b = await frappe.xcall("my_app.api.x", { a });
frm.set_value("b", b);
```

## Removed globals (v15)
`user` → `frappe.session.user`; `roles` → `frappe.user_roles`; `get_today()` → `frappe.datetime.get_today()`; `show_alert` → `frappe.show_alert`; `validated = false` → `frappe.validated = false`.
