=========================================================
🌉 ZF-BRIDGE — README.txt (Qbox / QBX Compatibility Bridge)
=========================================================

zf-bridge is a lightweight “compatibility layer” that routes common framework
events (especially QBX/Qbox notify events) into your ZF ecosystem resources.

It’s made to keep your server consistent by forwarding:
✅ Notifications → zf-notify (preferred)
✅ UI calls → zf-ui (preferred)
✅ Admin open commands → your ZF admin resources (event-based)
✅ Gang admin open commands → your gang system (event-based)

---------------------------------------------------------
📦 Requirements
---------------------------------------------------------
Required:
- None (bridge can run alone)

Recommended (based on your config):
- zf-notify  (preferred Notify)
- zf-ui      (preferred UI)

If you enable ox/qbx compatibility hooks, those target resources must exist
(or the bridge will simply do nothing if they’re not present / not running).

---------------------------------------------------------
📁 Recommended Folder Layout
---------------------------------------------------------
zf-bridge/
  fxmanifest.lua
  shared/
    config.lua
  client/
    client.lua
  server/
    server.lua

(Exact files may vary, but keep config in shared/.)

---------------------------------------------------------
⚙️ Installation
---------------------------------------------------------
1) Drop `zf-bridge` into your resources folder:
   resources/[zf]/zf-bridge

2) Ensure in your server.cfg (recommended order):
   ensure zf-notify
   ensure zf-ui
   ensure zf-bridge

3) Restart server (or start resources).

---------------------------------------------------------
🔧 Configuration (shared/config.lua)
---------------------------------------------------------
Your config:

Config = {}

-- =========================
-- What zf-bridge should do
-- =========================
Config.Bridge = {
  Notify = true,
  UI = true,
  Gang = true,
  Admin = true,
}

-- =========================
-- Preferred routing
-- =========================
Config.Prefer = {
  Notify = 'zf-notify', -- preferred notify resource
  UI = 'zf-ui',         -- preferred ui resource
}

-- =========================
-- Compatibility hooks
-- =========================
Config.Compat = {
  -- qbox/qbx notify events (enable these)
  QbxNotifyEvents = true,

  -- qb-core legacy notify events (off by default since you said qbox only)
  QbNotifyEvents = false,
}

-- Common event names to hook for qbx-only stacks
Config.QbxNotifyEventNames = {
  'qbx_core:client:notify',
  'qbx:client:notify',
  'qbx_core:notify',
}

-- Optional: if you want to support qb-core style notifies too
Config.QbNotifyEventNames = {
  'QBCore:Notify',
  'QBCore:Client:Notify',
}

-- =========================
-- Commands (optional)
-- =========================
Config.Commands = {
  Enable = true,
  AdminOpen = 'zfadmin',      -- opens your admin menu (event-based)
  GangAdminOpen = 'zfgangs',  -- opens gang admin (event-based)
}

-- These are the events zf-bridge will trigger for open commands.
-- Keep these stable across your resources.
Config.OpenEvents = {
  AdminMenu = 'zf-adminmenu:client:open',
  GangAdmin = 'zf-gangsystem:client:openStaff', -- you can add this event in your gangsystem
}

-- =========================
-- Type mapping
-- =========================
Config.TypeMap = {
  ['primary'] = 'info',
  ['inform']  = 'info',
  ['info']    = 'info',
  ['success'] = 'success',
  ['error']   = 'error',
  ['warning'] = 'warning',
  ['warn']    = 'warning',
}

---------------------------------------------------------
🧠 What Each Setting Does
---------------------------------------------------------

✅ Config.Bridge
Turn bridge modules on/off.
- Notify: forward notify events into your preferred notify resource
- UI: forward UI usage into your preferred UI resource (if implemented)
- Gang: enables gang admin open command/event routing
- Admin: enables admin menu open command/event routing

⭐ Config.Prefer
Tells the bridge which ZF resources to use first.
- Notify = 'zf-notify'
- UI     = 'zf-ui'

🔌 Config.Compat
Enable compatibility hooks for external/framework events.
- QbxNotifyEvents: hooks QBX/Qbox notify events (recommended for qbox)
- QbNotifyEvents: hooks old QB-Core notify events (optional)

📌 Config.*NotifyEventNames
Lists the event names zf-bridge will listen for.
When a hooked event fires, zf-bridge remaps the type using Config.TypeMap
and forwards it to your preferred notify resource.

⌨️ Config.Commands
If enabled, adds simple commands to open your ZF menus by triggering events.
- /zfadmin  → triggers Config.OpenEvents.AdminMenu
- /zfgangs  → triggers Config.OpenEvents.GangAdmin

📣 Config.OpenEvents
These are the events zf-bridge triggers when commands are used.
Keep them stable across your ZF resources.

🧾 Config.TypeMap
Normalizes “framework-style” notify types into ZF types:
- primary / inform → info
- warn → warning
- success / error / warning / info passthrough

---------------------------------------------------------
🔔 Notify Bridging (QBX / Qbox)
---------------------------------------------------------
If enabled:
Config.Bridge.Notify = true
Config.Compat.QbxNotifyEvents = true

zf-bridge will listen to these events:
- qbx_core:client:notify
- qbx:client:notify
- qbx_core:notify

…and forward them into:
Config.Prefer.Notify  (default: zf-notify)

So scripts that use QBX notify events will still display using your ZF notify.

Optional QB-Core support:
If you ever want QB-Core legacy scripts supported, set:
Config.Compat.QbNotifyEvents = true
and zf-bridge will also hook:
- QBCore:Notify
- QBCore:Client:Notify

---------------------------------------------------------
🖥️ UI Bridging (ZF UI)
---------------------------------------------------------
If enabled:
Config.Bridge.UI = true

zf-bridge routes UI interactions into:
Config.Prefer.UI (default: zf-ui)

This helps keep your server consistent by standardizing prompts/confirmations/
inputs (depending on what your bridge implements).

---------------------------------------------------------
🛡️ Admin & Gang Commands
---------------------------------------------------------
If enabled:
Config.Commands.Enable = true

Commands:
- /zfadmin  → triggers: zf-adminmenu:client:open
- /zfgangs  → triggers: zf-gangsystem:client:openStaff

Important:
- These are EVENT-BASED hooks.
- Your admin menu / gang system must register these events on the client.

---------------------------------------------------------
✅ Best Practices
---------------------------------------------------------
✅ Keep zf-bridge lightweight (only enable what you actually use)
✅ Prefer routing all notifications through zf-notify for consistency
✅ Keep Config.OpenEvents stable so other ZF scripts can rely on them
✅ If you enable legacy QB-Core hooks, keep TypeMap updated for weird types

---------------------------------------------------------
🛠️ Troubleshooting
---------------------------------------------------------

❌ “No notifications show”
✅ Ensure zf-bridge is started:
   ensure zf-bridge
✅ Ensure your preferred notify resource is started:
   ensure zf-notify
✅ Confirm compatibility hooks are enabled:
   Config.Compat.QbxNotifyEvents = true
✅ Confirm the script you’re using is actually firing one of the event names
   in Config.QbxNotifyEventNames

❌ “Commands don’t open menus”
✅ Ensure Config.Commands.Enable = true
✅ Confirm you are using the correct command names:
   /zfadmin
   /zfgangs
✅ Confirm your target resources register these events:
   zf-adminmenu:client:open
   zf-gangsystem:client:openStaff

❌ “Type/colors wrong”
✅ Update Config.TypeMap to include the types your scripts are sending

---------------------------------------------------------
📜 Credits / License
---------------------------------------------------------
ZF Bridge by your ZF ecosystem.
Customize freely for your server branding. ✨
