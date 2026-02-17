# OpenStack Database Janitor (os_janitor)

An Ansible role to automate database cleanup (purging) for OpenStack services running in Docker containers (typically Kolla/Kolla-Ansible deployments).

This role installs a systemd timer and service that periodically runs cleanup commands (like `nova-manage db archive_deleted_rows` and `purge`) to prevent database bloat and improve performance.

## Features

*   **Supported Services:** Nova (API & Conductor), Cinder, Heat, Glance, Masakari.
*   **Secure by Default:** All cleaners are disabled by default. You must explicitly enable them using `os_janitor_enable_<component>: true`.
*   **HA Safe:** The cleanup job runs only on **one node** of the control plane (the first node in the specified group) to avoid database locks and race conditions, even if the role is applied to all controllers.
*   **Logging:** Detailed execution logs are stored in `/var/log/openstack-janitor/janitor.log`.
*   **Configurable:** Full control over retention period, schedule, and batch sizes.

## Requirements

*   Target hosts must be running OpenStack services in Docker containers (Kolla/Kolla-Ansible).
*   `docker` CLI must be available on the target host and accessible by root.
*   `systemd` is required for scheduling the janitor service.

## Role Variables

The following variables can be configured. See `defaults/main.yml` for the full list.

| Variable | Default | Description |
|----------|---------|-------------|
| `os_janitor_retention_days` | `90` | Number of days to keep deleted data. Data older than this will be purged. |
| `os_janitor_schedule` | `"*-*-* 03:00:00"` | Systemd OnCalendar schedule format. Default is daily at 3:00 AM. |
| `os_janitor_control_group` | `"control"` | The Ansible inventory group name for your control plane nodes. **Must match your inventory.** |
| `os_janitor_max_rows` | `1000` | Batch size for row deletion (prevents DB locks). Used by Nova, Glance, Masakari. |
| `os_janitor_max_stacks` | `10` | Batch size for Heat stack purging. |
| `os_janitor_log_dir` | `/var/log/openstack-janitor` | Directory for execution logs. |
| `os_janitor_run_now` | `false` | If `true`, runs the janitor immediately after deployment (useful for testing). |

### Enabling Components

By default, no cleanup actions are performed. You must set the following flags to `true` in your inventory or `group_vars`:

*   `os_janitor_enable_nova_api`: Enable cleanup for Nova API DB
*   `os_janitor_enable_nova_conductor`: Enable cleanup for Nova Conductor DB (Cell DB)
*   `os_janitor_enable_cinder`: Enable cleanup for Cinder
*   `os_janitor_enable_heat`: Enable cleanup for Heat
*   `os_janitor_enable_glance`: Enable cleanup for Glance
*   `os_janitor_enable_masakari`: Enable cleanup for Masakari

## Dependencies

None.

## Example Playbook

```yaml
- hosts: control
  become: true
  roles:
    - role: barozek.os_janitor
      vars:
        # Retention policy: Keep deleted data for 30 days
        os_janitor_retention_days: 30
        
        # Schedule: Run every Sunday at 2:00 AM
        os_janitor_schedule: "Sun *-*-* 02:00:00"
        
        # Enable specific components
        os_janitor_enable_nova_api: true
        os_janitor_enable_nova_conductor: true
        os_janitor_enable_cinder: true
        os_janitor_enable_glance: true
        
        # Keep Heat and Masakari disabled (implicit default)
        # os_janitor_enable_heat: false
```

## License

MIT

## Author Information

Created by barozek.
For more information, visit the [GitHub repository](https://github.com/barozek90/os_janitor).
