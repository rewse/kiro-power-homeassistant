# Home Assistant MCP Server Tools Reference

## MCP Server: homeassistant

**Package:** `ha-mcp@latest`
**Connection:** uvx-based MCP server
**Authentication:** Home Assistant long-lived access token

**Configuration Required:**
- `HOMEASSISTANT_URL` - Your Home Assistant URL (e.g., `http://homeassistant.local:8123`)
- `HOMEASSISTANT_TOKEN` - Long-lived access token

## Available Tools (82 tools)

### Search & Discovery
- `ha_search_entities` - Fuzzy entity search
- `ha_deep_search` - Deep config search
- `ha_get_overview` - Get system overview
- `ha_get_state` - Get entity state

### Service & Device Control
- `ha_call_service` - Call service
- `ha_bulk_control` - Bulk device control
- `ha_get_operation_status` - Get operation status
- `ha_get_bulk_status` - Get bulk status
- `ha_list_services` - List services

### Automations
- `ha_config_get_automation` - Get automation
- `ha_config_set_automation` - Set automation
- `ha_config_remove_automation` - Remove automation

### Scripts
- `ha_config_get_script` - Get script
- `ha_config_set_script` - Set script
- `ha_config_remove_script` - Remove script

### Helper Entities
- `ha_config_list_helpers` - List helpers
- `ha_config_set_helper` - Set helper
- `ha_config_remove_helper` - Remove helper

### Dashboards
- `ha_config_list_dashboards` - List dashboards
- `ha_config_get_dashboard` - Get dashboard
- `ha_config_set_dashboard` - Set dashboard
- `ha_config_update_dashboard_metadata` - Update dashboard metadata
- `ha_config_delete_dashboard` - Delete dashboard
- `ha_get_dashboard_guide` - Get dashboard guide
- `ha_get_card_types` - Get card types
- `ha_get_card_documentation` - Get card documentation

### Areas & Floors
- `ha_config_list_areas` - List areas
- `ha_config_set_area` - Set area
- `ha_config_remove_area` - Remove area
- `ha_config_list_floors` - List floors
- `ha_config_set_floor` - Set floor
- `ha_config_remove_floor` - Remove floor

### Labels
- `ha_config_list_labels` - List labels
- `ha_config_get_label` - Get label
- `ha_config_set_label` - Set label
- `ha_config_remove_label` - Remove label
- `ha_assign_label` - Assign label

### Zones
- `ha_list_zones` - List zones
- `ha_create_zone` - Create zone
- `ha_update_zone` - Update zone
- `ha_delete_zone` - Delete zone

### Groups
- `ha_config_list_groups` - List groups
- `ha_config_set_group` - Set group
- `ha_config_remove_group` - Remove group

### Todo Lists
- `ha_list_todo_lists` - List todo lists
- `ha_get_todo_items` - Get todo items
- `ha_add_todo_item` - Add todo item
- `ha_update_todo_item` - Update todo item
- `ha_remove_todo_item` - Remove todo item

### Calendar
- `ha_config_get_calendar_events` - Get calendar events
- `ha_config_set_calendar_event` - Set calendar event
- `ha_config_remove_calendar_event` - Remove calendar event

### Blueprints
- `ha_list_blueprints` - List blueprints
- `ha_get_blueprint` - Get blueprint
- `ha_import_blueprint` - Import blueprint

### Device Registry
- `ha_list_devices` - List devices
- `ha_get_device` - Get device
- `ha_update_device` - Update device
- `ha_remove_device` - Remove device
- `ha_rename_entity` - Rename entity

### ZHA & Integrations
- `ha_get_zha_devices` - Get ZHA devices
- `ha_get_entity_integration_source` - Get entity integration source

### Add-ons
- `ha_list_addons` - List add-ons
- `ha_list_available_addons` - List available add-ons

### Camera
- `ha_get_camera_image` - Get camera image

### History & Statistics
- `ha_get_history` - Get history
- `ha_get_statistics` - Get statistics

### Automation Traces
- `ha_get_automation_traces` - Get automation traces

### System & Updates
- `ha_check_config` - Check config
- `ha_restart` - Restart
- `ha_reload_core` - Reload core
- `ha_get_system_info` - Get system info
- `ha_get_system_health` - Get system health
- `ha_list_updates` - List updates
- `ha_get_release_notes` - Get release notes
- `ha_get_system_version` - Get system version

### Backup & Restore
- `ha_backup_create` - Create backup
- `ha_backup_restore` - Restore backup

### Utility
- `ha_get_logbook` - Get logbook
- `ha_eval_template` - Evaluate template
- `ha_get_domain_docs` - Get domain docs
- `ha_list_integrations` - List integrations

## Tool Usage Guidelines

### When to Use Each Tool Category

**Search & Discovery:**
Use when you need to find entities or understand the system state. Start with `ha_search_entities` for fuzzy matching, use `ha_deep_search` for comprehensive config searches.

**Service & Device Control:**
Use `ha_call_service` for single operations, `ha_bulk_control` for multiple devices at once. Always check status with `ha_get_operation_status` or `ha_get_bulk_status` after operations.

**Automations:**
Use `ha_config_get_automation` to read existing automations before modifying. Use `ha_config_set_automation` to create or update. Always validate automation structure before setting.

**Scripts:**
Similar to automations but for reusable sequences. Use `ha_config_get_script` to read, `ha_config_set_script` to create/update.

**Dashboards:**
Use `ha_config_list_dashboards` to see available dashboards, `ha_config_get_dashboard` to read current config, `ha_config_set_dashboard` to update.

**Device Registry:**
Use `ha_list_devices` to discover devices, `ha_get_device` for details, `ha_update_device` to modify metadata.

**History & Statistics:**
Use `ha_get_history` for time-series data, `ha_get_statistics` for aggregated metrics.

**Automation Traces:**
Use `ha_get_automation_traces` when debugging automations that don't work as expected. Traces show execution history and why automations triggered or didn't trigger.

## Common Patterns

### Pattern 1: Safe Entity Control
1. Search for entity: `ha_search_entities`
2. Get current state: `ha_get_state`
3. Call service: `ha_call_service`
4. Verify result: `ha_get_operation_status`

### Pattern 2: Automation Creation
1. Check existing automations: `ha_config_get_automation`
2. Design automation structure
3. Set automation: `ha_config_set_automation`
4. Test and verify with traces: `ha_get_automation_traces`

### Pattern 3: Dashboard Customization
1. List dashboards: `ha_config_list_dashboards`
2. Get current dashboard: `ha_config_get_dashboard`
3. Modify dashboard structure
4. Update dashboard: `ha_config_set_dashboard`

### Pattern 4: Bulk Operations
1. Search for target entities: `ha_search_entities`
2. Execute bulk control: `ha_bulk_control`
3. Check bulk status: `ha_get_bulk_status`

## Error Handling

When tools return errors:
1. Check entity IDs are correct (use `ha_search_entities` to verify)
2. Verify service names with `ha_list_services`
3. Ensure proper authentication (check token validity)
4. Review automation traces for automation issues
5. Check system health with `ha_get_system_health`
