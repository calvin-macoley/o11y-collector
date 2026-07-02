alloy_install
=============

Installs Grafana Alloy via the `grafana.grafana.alloy` collection role.

Alloy configuration
-------------------

The Alloy configuration is not written inline in the tasks. It lives in
[`templates/config.alloy.j2`](templates/config.alloy.j2), which is rendered on
the control node with a template lookup and passed to the collection role as
the `alloy_config` variable:

```yaml
alloy_config: "{{ lookup('ansible.builtin.template', 'config.alloy.j2') }}"
```

Because the lookup runs inside this role, the bare filename resolves to this
role's `templates/` directory. The template has access to all inventory
variables, so per-environment values (e.g. the remote-write endpoint from
`group_vars`) are interpolated at render time.

To change the Alloy pipeline, edit `templates/config.alloy.j2` — not the task
file. Anything that must stay a literal `{{ }}` in the final Alloy config has
to be escaped in the template as `{{ '{{' }}`.
