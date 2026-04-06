# Sophos SFOS 22 Graylog Extractor

Graylog extractor configuration for Sophos Firewall running SFOS 22, parsing syslog input into discrete searchable fields.

## Overview

This repository contains a Graylog extractor JSON for ingesting and parsing syslog output from Sophos Firewall (SFOS 22). The extractor targets a Syslog UDP input and parses 66 fields from SFOS log data including firewall rules, network traffic, application identification, web filtering, and connection metadata. All extractors are conditioned on the Sophos device name to prevent interference with other syslog sources on the same input. Fields are derived from real SFOS 22 log data and reflect current field names -- older SFOS field name variants have been intentionally excluded.

## Tested Against

- **Graylog:** 7.0.6+711d207
- **Sophos SFOS:** 22
- **Syslog format:** Standard Syslog (UDP)

## Fields Parsed

| Category | Fields |
|---|---|
| Identity / Envelope | `device_name`, `device_model`, `device_serial_id`, `timestamp`, `log_id`, `log_type`, `log_component`, `log_subtype`, `log_version`, `severity` |
| Rule / Policy | `fw_rule_id`, `fw_rule_name`, `fw_rule_section`, `fw_rule_type`, `nat_rule_id`, `nat_rule_name`, `web_policy_id`, `ips_policy_id`, `app_filter_policy_id` |
| Network | `src_ip`, `dst_ip`, `src_port`, `dst_port`, `protocol`, `src_mac`, `dst_mac`, `src_country`, `dst_country`, `src_zone`, `dst_zone`, `src_zone_type`, `dst_zone_type`, `src_trans_ip`, `in_interface`, `out_interface`, `in_display_interface`, `out_display_interface`, `ether_type` |
| Traffic Stats | `bytes_sent`, `bytes_received`, `packets_sent`, `packets_received`, `duration`, `con_id`, `con_event` |
| Application | `app_name`, `app_risk`, `app_technology`, `app_category`, `app_resolved_by`, `app_is_cloud`, `gw_id_request`, `gw_name_request` |
| Web / Content | `url`, `domain`, `http_category`, `http_category_type`, `http_status`, `used_quota`, `activity_name` |
| Other | `hb_status`, `qualifier`, `classification`, `exceptions`, `message`, `log_occurrence` |

All fields are stored as strings, including numeric fields such as `src_port`, `dst_port`, `bytes_sent`, and `bytes_received`.

## Prerequisites

### Sophos Firewall

1. Go to **System services > Log settings** in the SFOS web GUI.
2. Click **Add** to configure a syslog server.
3. Set the following:
   - **IP/Hostname:** Your Graylog server IP
   - **Port:** Your chosen UDP port (e.g. `5140`)
   - **Protocol:** UDP
   - **Log format:** Standard Syslog
4. Enable the log categories you want to collect (Firewall, Web, IPS, etc.).
5. Click **Save**.

### Graylog

1. Go to **System > Inputs**.
2. Select **Syslog UDP** from the dropdown and click **Launch new input**.
3. Configure the input:
   - **Title:** Sophos SFOS Syslog (or your preferred name)
   - **Bind address:** `0.0.0.0`
   - **Port:** Match the port configured on the Sophos side
   - **Allow override date:** Checked
   - **Store full message:** Checked
4. Click **Save** and confirm the input shows a green **RUNNING** status.

## Importing the Extractor

1. Go to **System > Inputs** in Graylog.
2. Find your Sophos syslog input and click **Manage Extractors**.
3. Click **Actions > Import extractors**.
4. Paste the contents of `Sophos_SFOS22_Graylog_Extractor.json` into the text field.
5. Click **Import**.

After importing, trigger a log event on the firewall (a denied connection, an auth event, etc.) and open a message in **Search**. Expand the message detail view and confirm the parsed fields are present alongside the raw `message` field.

## Important: device_name Condition

Every extractor in this file is conditioned on `device_name="sophosfw"`. This means extractors will only fire on messages that contain that exact string. If your Sophos firewall is configured with a different device name, the extractors will silently skip all messages and no fields will be parsed.

**To verify your device name**, expand any raw message from your firewall in Graylog Search and look for the `device_name` field value at the start of the message body.

**To update the condition**, open `Sophos_SFOS22_Graylog_Extractor.json` in a text editor and do a find-and-replace:

- Find: `device_name="sophosfw"`
- Replace: `device_name="your_device_name"`

Then import the updated file. Alternatively, after importing you can edit each extractor individually in Graylog under **Manage Extractors** and update the condition value field.

## License

MIT
