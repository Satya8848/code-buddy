# Target Layout & Refactor Patterns

```
my_app/my_app/
  hooks.py                    # wiring only
  api/<domain>.py             # @frappe.whitelist endpoints: validate input, check perms, call services
  services/<domain>.py        # business logic
  events/<doctype_snake>.py   # doc_events handlers: def validate(doc, method=None): ...
  overrides/<doctype_snake>.py# class CustomSalesInvoice(SalesInvoice): ...
  tasks/<frequency_or_domain>.py
  utils/<topic>.py            # dates.py, numbers.py — not one giant utils.py
  patches/v1_0/<patch_name>.py
  public/js/<doctype_snake>.js
  fixtures/
```

## hooks.py doc_events — before
```python
doc_events = {
    "Sales Invoice": {
        "validate": "my_app.my_app.custom.si_validate",
        "on_submit": "my_app.api.si_submit",
    }
}
```
## after
```python
doc_events = {
    "Sales Invoice": {
        "validate": "my_app.events.sales_invoice.validate",
        "on_submit": "my_app.events.sales_invoice.on_submit",
    }
}
```

## Thin controller pattern
```python
# dispatch_plan.py
from my_app.services import dispatch

class DispatchPlan(Document):
    def validate(self):
        dispatch.validate_vehicle_capacity(self)
        dispatch.set_route_totals(self)

    def on_submit(self):
        frappe.enqueue(
            "my_app.services.dispatch.create_delivery_notes",
            queue="long", job_id=f"dn-{self.name}", deduplicate=True,
            enqueue_after_commit=True, plan_name=self.name,
        )
```

## Backward-compatible API move
```python
# my_app/api/__init__.py  (keeps old dotted path my_app.api.get_open_plans working)
from my_app.api.dispatch import get_open_plans as _get_open_plans

@frappe.whitelist()
def get_open_plans(**kwargs):  # Deprecated: use my_app.api.dispatch.get_open_plans
    return _get_open_plans(**kwargs)
```
Note: a module and a package can't share a name (`api.py` vs `api/`). When converting `api.py` into an `api/` package, move the wrappers into `api/__init__.py` so the old dotted path `my_app.api.get_open_plans` keeps working.
