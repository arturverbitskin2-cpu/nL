local function _v(x) return x end local V = {} V.PC = {} V.WR = 0 local function nn(a, b) if a == nil then return b end return a end local _tostring = tostring local _tonumber = tonumber local _type = type
local function type(v) if v == nil then return "nil" end return _type(v) end local function tostring(v) if v == nil then return "nil" end local ok, r = pcall(_tostring, v) if ok then return r end return "?" end
local function tonumber(v) if v == nil then return nil end local ok, r = pcall(_tonumber, v) if ok then return r end return nil end
V.CLK = { src = "time", acc = 0, off = 0, t0 = false, a0 = 0, checked = false, ratio = 0 } V.clk_raw = function(src) if src == "acc" then return V.CLK.acc end local f = time if src == "osclock" then f = os.clock end
local ok, v = pcall(f) if ok and type(v) == "number" then return v end return false end V.clk_switch = function(to) local cur = (V.clk_raw(V.CLK.src) or V.CLK.acc) + V.CLK.off local nv = V.clk_raw(to) if not nv then
to = "acc" nv = V.CLK.acc end V.CLK.src = to V.CLK.off = cur - nv V.CLK.t0 = false end V.now_s = function() local v = V.clk_raw(V.CLK.src) if not v then V.clk_switch((V.CLK.src == "time") and "osclock" or "acc")
v = V.clk_raw(V.CLK.src) or V.CLK.acc end return v + V.CLK.off end V.clk_frame = function(dt) local okf, ft = pcall(globals.frametime) local d = (okf and type(ft) == "number" and ft > 0 and ft < 1) and ft
or ((type(dt) == "number" and dt > 0 and dt < 1) and dt or 0.016) V.CLK.acc = V.CLK.acc + d end client.set_event_callback("paint", function(dt) V.clk_frame(dt) end) V.reported = {} V.report = function(name, err)
if V.reported[name] then return end V.reported[name] = true pcall(client.color_log, 255, 120, 0, "[NP] API fail: " .. name .. " -> " .. tostring(err)) end V.api_ret = function(name, ok, r1, r2)
if not ok then V.report(name, r1) return false end return true, r1, r2 end V.api = function(name, fn, a1, a2, a3) if type(fn) ~= "function" then V.report(name, "nil (функции нет)") return false end
if a1 == nil then return V.api_ret(name, pcall(fn)) end if a2 == nil then return V.api_ret(name, pcall(fn, a1)) end if a3 == nil then return V.api_ret(name, pcall(fn, a1, a2)) end
return V.api_ret(name, pcall(fn, a1, a2, a3)) end V.log = function(r, g, b, msg) if not pcall(client.color_log, r, g, b, msg) then print(msg) end end do local fails = "" local function t(name, fn)
local ok, res = pcall(fn) if not (ok and res == true) then fails = fails .. name .. " " end end t("call-multi-assign", function() local x = _v(0) local y = _v(0) x, y = (function() return 1, 2 end)()
return x == 1 and y == 2 end) t("nil-upvalue", function() local z = nil local function set() z = 1 end set() return z == 1 end) t("unroll-closure", function() local fs = {}
for i = 1, 3 do fs[i] = function() return i end end return fs[1]() == 1 and fs[3]() == 3 end) t("assoc-minus", function() return (10 - 3 - 2) == 5 end) t("tostring-nil", function() return tostring(nil) == "nil" end)
if fails ~= "" then V.log(150, 150, 170, "[NP] interpreter quirks active: " .. fails .. "(workarounds on)") end end V.capture_item = _v(nil) V.ui_open = _v(false) V.ui_open_t = _v(0) V.ui_hint_done = _v(false)
V.reveal_t = _v(false) V.UI_TABS = {} V.UI_TN = _v(0) V.UI_COLOR_NAMES = { "Blue", "Cyan", "Purple", "Pink", "Red", "Orange", "Gold", "Mint", "Green", "White" } V.UI_COLORS = {
Blue = color(70, 160, 255), Cyan = color(0, 220, 255), Purple = color(150, 90, 255), Pink = color(255, 90, 190), Red = color(255, 80, 90), Green = color(80, 230, 140), White = color(235, 240, 255),
Orange = color(255, 140, 50), Gold = color(255, 196, 60), Mint = color(90, 240, 200), } V.EGG = { clicks = {}, honey = false, rainbow_until = 0, buf = "", king = 0, king_t = -10 } V.HONEY_COL = color(255, 184, 28)
V.hsv = function(h, sat, val) local i = math.floor(h * 6) local f = (h * 6) - i local p = val * (1 - sat) local q = val * (1 - (f * sat)) local t = val * (1 - ((1 - f) * sat)) local m = i % 6 local c = { val, t, p }
if m == 1 then c = { q, val, p } elseif m == 2 then c = { p, val, t } elseif m == 3 then c = { p, q, val } elseif m == 4 then c = { t, p, val } elseif m == 5 then c = { val, p, q } end
return color(math.floor(c[1] * 255), math.floor(c[2] * 255), math.floor(c[3] * 255)) end V.ui_count = function(t) local n = 0 for _ in ipairs(t) do n = n + 1 end return n end V.w_get = function(self) return self.v end
V.w_set = function(self, v, silent) self.v = v if not silent and self.cb then pcall(self.cb, v) end end V.w_cb = function(self, f) self.cb = f end V.w_vis = function(self, b) local h = not b if self.hidden ~= h then
self.hidden = h if self.sec then self.sec.lay = nil end end end V.c_get = function(self) if self.egg then local now = V.now_s() if V.EGG.rainbow_until > now then return V.hsv((now * 0.25) % 1, 0.65, 1) end
if V.EGG.honey then return V.HONEY_COL end end return V.UI_COLORS[self.v] or V.UI_COLORS.Blue end V.ui_add = function(sec, w) w.get = V.w_get w.set = V.w_set w.set_callback = V.w_cb w.set_visible = V.w_vis w.sec = sec
w.hidden = false sec.n = sec.n + 1 sec.items[sec.n] = w return w end V.SEC = {}
function V.SEC.new_toggle(s, name, def, cb) return V.ui_add(s, { t = "toggle", name = name, v = def and true or false, anim = def and 1 or 0, cb = cb }) end
function V.SEC.new_slider(s, name, mn, mx, def, dec, suf, cb) return V.ui_add(s, { t = "slider", name = name, min = mn, max = mx, v = def or mn, suf = suf or "", cb = cb }) end
function V.SEC.new_combobox(s, name, opts, def, cb) return V.ui_add(s, { t = "combo", name = name, opts = opts, on = V.ui_count(opts), v = def or opts[1], cb = cb }) end
function V.SEC.new_textbox(s, name, ph, def, cb) return V.ui_add(s, { t = "label", name = name .. ":  " .. tostring(def or ""), v = def or "", cb = cb }) end
function V.SEC.new_button(s, name, cb) return V.ui_add(s, { t = "button", name = name, v = false, cb = cb }) end
function V.SEC.new_keybind(s, name, def, cb) return V.ui_add(s, { t = "key", name = name, v = def or "None", cb = cb }) end function V.SEC.new_label(s, txt) return V.ui_add(s, { t = "label", name = txt, v = txt }) end
function V.SEC.new_colorpicker(s, name, def, cb) local w = V.ui_add(s, { t = "combo", name = name, opts = V.UI_COLOR_NAMES, on = V.ui_count(V.UI_COLOR_NAMES), v = "Blue", cb = cb, egg = true }) w.get = V.c_get return w
end V.ui_new_tab = function() return { new_section = function(_, name) local sec = { name = name, items = {}, n = 0 } for k, f in pairs(V.SEC) do sec[k] = f end V.UI_TN = V.UI_TN + 1 V.UI_TABS[V.UI_TN] = sec return sec
end } end V.tab = V.ui_new_tab() V.s_aa = V.tab:new_section("Anti-Aim", "Left") V.s_st = V.tab:new_section("AA States", "Left") V.s_def = V.tab:new_section("Defensive", "Left")
V.s_hf = V.tab:new_section("Hard Flick", "Left") V.s_res = V.tab:new_section("Resolver", "Right") V.s_rg = V.tab:new_section("Rage", "Right") V.s_vis = V.tab:new_section("Visuals", "Right")
V.s_pl = V.tab:new_section("Players", "Right") V.s_ext = V.tab:new_section("Extras", "Right") V.s_gr = V.tab:new_section("Grenades", "Right") V.s_keys = V.tab:new_section("Binds", "Right")
V.s_cfg = V.tab:new_section("Config", "Right") V.aa_on = V.s_aa:new_toggle("Enable AA", true) V.yaw_mode = V.s_aa:new_combobox("Yaw Modifier", { "Mixed", "Jitter", "3-Way", "Random", "Sway" }, "Mixed")
V.yaw_range = V.s_aa:new_slider("Modifier Range", 0, 90, 42, 0, "°") V.yaw_add = V.s_aa:new_slider("Yaw Offset", -45, 45, 0, 0, "°") V.desync_lim = V.s_aa:new_slider("Left Limit", 0, 86, 86, 0, "°")
V.desync_lim.key = "Desync Limit" V.desync_lim_r = V.s_aa:new_slider("Right Limit", 0, 86, 86, 0, "°") V.lim_jit = V.s_aa:new_combobox("Limit Jitter", { "Off", "Swap", "Random" }, "Off")
V.lim_jit_spd = V.s_aa:new_slider("Limit Jitter Speed", 1, 16, 4, 0, " t") V.inv_mode = V.s_aa:new_combobox("Inverter", { "Jitter", "Random", "Static" }, "Jitter")
V.anti_brute = V.s_aa:new_toggle("Anti-Bruteforce", true) V.brute_mode = V.s_aa:new_combobox("Anti-Brute Mode", { "Random", "Stages" }, "Random") V.brute_reset = V.s_aa:new_slider("Stage Reset", 0, 60, 20, 0, " s")
V.mod_hold = V.s_aa:new_slider("Modifier Hold", 1, 8, 1, 0, " t") V.mod_hold_rnd = V.s_aa:new_toggle("Randomize Hold", false) V.ST_NAMES = { "Standing", "Moving", "Crouching", "In Air", "Air Crouch" }
V.ST_KEYS = { "standing", "moving", "crouching", "in air", "air crouch" } V.sa_on = V.s_st:new_toggle("Enable AA States", false) V.sa_edit = V.s_st:new_combobox("Edit State", V.ST_NAMES, "Standing") V.SA = {}
V.SA_BY = {} for i = 1, 5 do local nm = V.ST_NAMES[i] local c = { ov = V.s_st:new_toggle("Override Global", false), mode = V.s_st:new_combobox("Yaw Modifier", { "Mixed", "Jitter", "3-Way", "Random", "Sway" }, "Mixed"),
range = V.s_st:new_slider("Modifier Range", 0, 90, 42, 0, "°"), add = V.s_st:new_slider("Yaw Offset", -45, 45, 0, 0, "°"), lim = V.s_st:new_slider("Left Limit", 0, 86, 86, 0, "°"),
rlim = V.s_st:new_slider("Right Limit", 0, 86, 86, 0, "°"), inv = V.s_st:new_combobox("Inverter", { "Jitter", "Random", "Static" }, "Jitter"), def = V.s_st:new_toggle("Defensive Flicks", true), }
c.ov.key = nm .. " override" c.mode.key = nm .. " modifier" c.range.key = nm .. " range" c.add.key = nm .. " offset" c.lim.key = nm .. " desync" c.rlim.key = nm .. " rdesync" c.inv.key = nm .. " inverter"
c.def.key = nm .. " defensive" V.SA[i] = c V.SA_BY[V.ST_KEYS[i]] = c end V.sa_refresh = function() local e = V.sa_edit:get() for i = 1, 5 do local on = (V.ST_NAMES[i] == e)
for _, w in pairs(V.SA[i]) do w:set_visible(on) end end end V.sa_edit:set_callback(V.sa_refresh) V.sa_refresh() V.def_on = V.s_def:new_toggle("Defensive Flicks", true)
V.def_interval = V.s_def:new_slider("Flick Interval", 4, 32, 9, 0, " t") V.def_len = V.s_def:new_slider("Flick Length", 1, 4, 2, 0, " t") V.def_shift = V.s_def:new_toggle("Tickbase Shift", false)
V.def_onshot = V.s_def:new_toggle("Flick on Own Shot", true) V.def_air_lc = V.s_def:new_toggle("Break LC in Air", true) V.acc_shoot = V.s_def:new_toggle("Accurate Shooting", true)
V.acc_lag = V.s_def:new_slider("Lag While Shooting", 0, 6, 1, 0, " t") V.aa_shot = V.s_def:new_combobox("AA While Shooting", { "Keep", "Disable" }, "Disable") V.def_spin = V.s_def:new_toggle("Spin Pitch On Shot", false)
V.def_spin_ms = V.s_def:new_slider("Spin Duration", 100, 1000, 350, 0, " ms") V.def_trig = V.s_def:new_combobox("Flick Trigger", { "Timer", "On Peek", "Timer + Peek" }, "Timer")
V.def_peek_len = V.s_def:new_slider("Peek Flick Window", 2, 24, 8, 0, " t") V.hf_on = V.s_hf:new_toggle("Enable Hard Flick", false) V.hf_speed = V.s_hf:new_combobox("Speed Mode", { "Fixed", "Wave" }, "Fixed")
V.hf_ms = V.s_hf:new_slider("Flick Speed", 10, 200, 40, 0, " ms") V.hf_ms2 = V.s_hf:new_slider("Wave Fast Speed", 10, 200, 30, 0, " ms") V.hf_cycle = V.s_hf:new_slider("Wave Cycle", 1, 10, 2, 0, "x")
V.hf_a = V.s_hf:new_slider("Angle A", 0, 86, 86, 0, "°") V.hf_rnd_a = V.s_hf:new_toggle("Random A (0-86)", false) V.hf_b = V.s_hf:new_slider("Angle B", 0, 86, 0, 0, "°")
V.hf_rnd_b = V.s_hf:new_toggle("Random B (0-86)", false) V.hf_refresh = function() local wave = (V.hf_speed:get() == "Wave") V.hf_ms2:set_visible(wave) V.hf_cycle:set_visible(wave) end
V.hf_speed:set_callback(V.hf_refresh) V.hf_refresh() V.hf_state = _v(false) V.hf_last = _v(0) V.hf_cur = _v(86) V.res_on = V.s_res:new_toggle("Enable Lua Resolver", true)
V.res_mode = V.s_res:new_combobox("Resolver Mode", { "Built-in + Assist", "Lua Brute" }, "Built-in + Assist")
V.bi_cycle = V.s_res:new_combobox("Built-in Mode Switch", { "Off", "On Miss", "Every Second", "Per AA Type" }, "Off") V.res_cls = V.s_res:new_toggle("AA Classifier", true)
V.res_ptype = V.s_res:new_toggle("Per-Type Strategy", true) V.res_defbody = V.s_res:new_toggle("Body First vs Defensive", false) V.kill_builtin = V.s_res:new_toggle("Disable Built-in", true)
V.max_desync = V.s_res:new_slider("Max Enemy Desync", 30, 86, 86, 0, "°") V.jitter_det = V.s_res:new_toggle("Jitter Detection", true) V.remember_hit = V.s_res:new_toggle("Remember Hit Angle", true)
V.res_score = V.s_res:new_toggle("Angle Memory", true) V.res_dmgval = V.s_res:new_toggle("Damage-Validated Hits", true) V.res_noaa = V.s_res:new_toggle("AA-Off Detection", true)
V.res_anim = V.s_res:new_toggle("Anim Resolver", true) V.baim_after = V.s_res:new_slider("BAim After Misses", 0, 5, 2, 0, " miss") V.baim_mode = V.s_res:new_combobox("BAim Mode", { "Prefer", "Force" }, "Prefer")
V.hard_on = V.s_res:new_toggle("Remember Hard Targets", true) V.hard_after = V.s_res:new_slider("Hard After Misses", 1, 10, 3, 0, " total") V.sp_after = V.s_res:new_slider("Safe Point After", 0, 5, 2, 0, " miss")
V.enemy_pitch = V.s_res:new_combobox("Enemy Pitch", { "Off", "Auto", "Force 0" }, "Off") V.log_shots = V.s_res:new_toggle("Log Shots To Console", true)
V.hb_prio = V.s_rg:new_combobox("Hitbox Priority", { "Menu", "Body First", "Head First" }, "Menu") V.scope_wait = V.s_rg:new_toggle("Wait For Scope", false)
V.scope_delay = V.s_rg:new_slider("Scope Delay", 0, 300, 120, 0, " ms") V.hc_on = V.s_rg:new_toggle("Adaptive Hitchance", false) V.hc_base = V.s_rg:new_slider("Base Hitchance", 40, 95, 80, 0, "%")
V.hc_step = V.s_rg:new_slider("+ Per Spread Miss", 0, 15, 5, 0, "%") V.hc_max = V.s_rg:new_slider("Max Hitchance", 50, 100, 90, 0, "%") V.lag_on = V.s_rg:new_toggle("Enable Fake Lag", false)
V.lag_limit = V.s_rg:new_slider("Lag Limit", 1, 14, 14, 0, " t") V.lag_var = V.s_rg:new_slider("Lag Variability", 0, 100, 35, 0, "%") V.lag_adapt = V.s_rg:new_toggle("Adaptive Lag", true)
V.sp_force = V.s_rg:new_toggle("Force Safe Point", false) V.lethal_mode = V.s_rg:new_combobox("Lethal BAim", { "Off", "Prefer", "Force" }, "Prefer") V.smart_dt = V.s_rg:new_toggle("Smart DT", true)
V.dyn_md = V.s_rg:new_toggle("Dynamic Min Damage", true) V.mhc_on = V.s_rg:new_toggle("Movement Hitchance", true) V.mhc_walk = V.s_rg:new_slider("HC Walk", 40, 100, 75, 0, "%")
V.mhc_run = V.s_rg:new_slider("HC Run", 40, 100, 65, 0, "%") V.mhc_air = V.s_rg:new_slider("HC Air", 40, 100, 80, 0, "%") local SS_OK, SS_W, SS_H = pcall(render.screen_size)
V.sw = (SS_OK and type(SS_W) == "number") and SS_W or 1920 V.sh = (SS_OK and type(SS_H) == "number") and SS_H or 1080 V.wm_on = V.s_vis:new_toggle("Watermark", true)
V.wm_name = V.s_vis:new_textbox("Title", "NeverPeek", "NeverPeek") V.accent_col = V.s_vis:new_colorpicker("Accent Color", color(70, 160, 255))
V.font_sel = V.s_vis:new_combobox("Font", { "Gotham", "Builder Sans", "Montserrat", "Ubuntu", "Classic UI", "Plex" }, "Gotham") V.wm_stats = V.s_vis:new_toggle("Stats Panel", true)
V.show_binds = V.s_vis:new_toggle("Keybinds List", true) V.show_cross = V.s_vis:new_toggle("Crosshair Indicators", true) V.show_state = V.s_vis:new_toggle("Player State", true)
V.show_toasts = V.s_vis:new_toggle("Hit Logs", true) V.wm_drag = V.s_vis:new_toggle("Drag Watermark", true) V.show_adv = V.s_vis:new_toggle("Show Advanced", false) V.intro_on = V.s_vis:new_toggle("Intro Animation", true)
V.intro_dim = V.s_vis:new_toggle("Intro Dim + Vignette", true) V.perf_mode = V.s_vis:new_combobox("Performance", { "Quality", "Balanced", "Potato" }, "Quality")
V.cull_scope = V.s_vis:new_toggle("Cull Crosshair In Scope", false) V.cull_toast = V.s_vis:new_toggle("Cull Max 3 Hit Logs", false) V.prof_on = V.s_vis:new_toggle("Perf Profiler", false)
V.tips_lang = V.s_vis:new_combobox("Tips Language", { "EN", "RU" }, "EN") V.wm_x = V.s_vis:new_slider("Position X", 0, V.sw, 20, 0, "px") V.wm_y = V.s_vis:new_slider("Position Y", 0, V.sh, 70, 0, "px")
V.dt_bar = V.s_vis:new_toggle("DT Bar", true) V.tr_on = V.s_vis:new_toggle("Bullet Tracers", true) V.tr_time = V.s_vis:new_slider("Tracer Time", 1, 6, 2, 0, " s")
V.dt_x = V.s_vis:new_slider("DT Bar X", 0, V.sw, math.max(0, V.sw - 200), 0, "px") V.dt_y = V.s_vis:new_slider("DT Bar Y", 0, V.sh, math.floor(V.sh * 0.62), 0, "px") V.s_vis:new_button("Reset Stats", function()
_G.__rt_reset_stats() end) V.k_aa = V.s_keys:new_keybind("Anti-Aim Toggle", "G") V.k_pitch = V.s_keys:new_keybind("Pitch Up / Down", "F") V.k_inv = V.s_keys:new_keybind("Manual Inverter", "X")
V.k_menu = V.s_keys:new_keybind("Menu Key", "RightShift") V.s_keys:new_label("click a bind, then press a key") V.s_keys:new_label("Escape = cancel") V.ann_on = V.s_ext:new_toggle("Killstreak Announcer", true)
V.ann_hs = V.s_ext:new_toggle("Announce Headshots", false) V.ann_window = V.s_ext:new_slider("Multi-kill Window", 2, 15, 6, 0, "s")
V.ann_sound = V.s_ext:new_combobox("Announcer Sound", { "Off", "Neverlose", "Skeet", "Parry", "Coins", "Headshoot", "Minecraft OOF" }, "Neverlose") V.lab_on = V.s_ext:new_toggle("Enemy Labels", true)
V.ks_on = V.s_ext:new_toggle("Kill Say", false) V.ks_mode = V.s_ext:new_combobox("Kill Say On", { "Every Kill", "Headshots", "Streaks" }, "Every Kill")
V.ks_lang = V.s_ext:new_combobox("Kill Say Language", { "EN", "RU" }, "EN") V.ks_cd = V.s_ext:new_slider("Kill Say Cooldown", 2, 30, 6, 0, "s") V.ks_test = V.s_ext:new_button("Test Chat")
V.bee_on = V.s_ext:new_toggle("Bees", false) V.bee_style = V.s_ext:new_combobox("Bee Style", { "Emoji", "Drawn" }, "Emoji") V.bee_n = V.s_ext:new_slider("Bee Count", 1, 12, 6, 0, "")
V.bee_rad = V.s_ext:new_slider("Bee Orbit", 1, 8, 3, 1, " st") V.bee_size = V.s_ext:new_slider("Bee Size", 8, 40, 20, 0, "px") V.bee_attack = V.s_ext:new_toggle("Bees Swarm Kills", true)
V.bee_trail = V.s_ext:new_toggle("Pollen Trail", true) V.mus_on = V.s_ext:new_toggle("Intro Music", true) V.mp_v = V.s_ext:new_slider("Music Volume", 0, 100, 50, 0, "%")
V.gn_pred = V.s_gr:new_toggle("Grenade Prediction", true) V.gn_throw = V.s_gr:new_combobox("Throw Strength", { "Auto", "Full", "Medium", "Short" }, "Auto") V.gn_inh = V.s_gr:new_toggle("Velocity Inheritance", true)
V.gn_xray = V.s_gr:new_toggle("Draw Through Walls", true) V.gn_ground = V.s_gr:new_toggle("Zones Only On Ground", true) V.gn_lbl = V.s_gr:new_toggle("Damage Labels", true)
V.gn_live = V.s_gr:new_toggle("Track Thrown Grenades", true) V.gn_fire = V.s_gr:new_toggle("Molotov Fire Zones", true) V.gn_fire_t = V.s_gr:new_slider("Fire Lifetime", 1, 12, 7, 0, " s")
V.gn_fire_r = V.s_gr:new_slider("Fire Radius", 3, 20, 10, 0, " st") V.gn_he_r = V.s_gr:new_slider("HE Radius", 6, 40, 24, 0, " st") V.gn_he_d = V.s_gr:new_slider("HE Damage", 20, 200, 98, 0, "")
V.pl_on = V.s_pl:new_toggle("Players List", true) V.pl_dead = V.s_pl:new_toggle("Show Dead Players", true) V.pl_hl = V.s_pl:new_toggle("Highlight Target", true) V.miss_bd = V.s_pl:new_toggle("Miss Breakdown", true)
V.pl_x = V.s_pl:new_slider("List X", 0, V.sw, math.max(0, V.sw - 420), 0, "px") V.pl_y = V.s_pl:new_slider("List Y", 0, V.sh, math.floor(V.sh * 0.3), 0, "px") V.s_pl:new_label("drag the list while the menu is open")
V.cfg_slot = V.s_cfg:new_combobox("Config Slot", { "Slot 1", "Slot 2", "Slot 3", "Slot 4", "Slot 5" }, "Slot 1") V.cfg_status = V.s_cfg:new_label("status:  -") V.cfg_btn_s = V.s_cfg:new_button("Save Config")
V.cfg_btn_l = V.s_cfg:new_button("Load Config") V.cfg_btn_r = V.s_cfg:new_button("Reset To Default") V.cfg_btn_e = V.s_cfg:new_button("Save Code") V.cfg_btn_i = V.s_cfg:new_button("Load Code")
V.cfg_preset = V.s_cfg:new_combobox("AA Preset", { "Jitter Meta", "Static Desync", "Hard Flick", "Defensive Heavy" }, "Jitter Meta") V.cfg_btn_p = V.s_cfg:new_button("Apply Preset") V.s_hf.adv = true
V.ADV_W = { V.bi_cycle, V.enemy_pitch, V.lim_jit, V.mod_hold, V.mod_hold_rnd, V.cull_scope, V.cull_toast, V.prof_on }
V.HIDE_W = { V.jitter_det, V.wm_x, V.wm_y, V.dt_x, V.dt_y, V.pl_x, V.pl_y, V.cfg_slot, V.cfg_btn_s, V.cfg_btn_l } V.vis_refresh = function() local adv = V.show_adv:get()
for _, w in ipairs(V.ADV_W) do w:set_visible(adv) end for _, w in ipairs(V.HIDE_W) do w:set_visible(false) end V.lim_jit_spd:set_visible(adv and V.lim_jit:get() ~= "Off") local hc = V.hc_on:get()
V.hc_base:set_visible(hc); V.hc_step:set_visible(hc); V.hc_max:set_visible(hc) local mh = V.mhc_on:get() V.mhc_walk:set_visible(mh); V.mhc_run:set_visible(mh); V.mhc_air:set_visible(mh)
V.scope_delay:set_visible(V.scope_wait:get()) local lg = V.lag_on:get() V.lag_limit:set_visible(lg); V.lag_var:set_visible(lg); V.lag_adapt:set_visible(lg) for i = 1, V.UI_TN do V.UI_TABS[i].lay = nil end end
for _, w in ipairs({ V.show_adv, V.lim_jit, V.hc_on, V.mhc_on, V.scope_wait, V.lag_on }) do w:set_callback(V.vis_refresh) end V.vis_refresh()
for _, w in ipairs({ V.cfg_slot, V.cfg_status, V.cfg_btn_s, V.cfg_btn_l, V.cfg_btn_r, V.cfg_btn_e, V.cfg_btn_i, V.cfg_preset, V.cfg_btn_p }) do w.nosave = true end pcall(render.clear)
V.stats = { hits = 0, misses = 0, spread = 0, res_miss = 0, occ = 0, dead = 0, other = 0, kills = 0, taken = 0, taken_hs = 0, deaths = 0, streak = 0, best_streak = 0 } V.C_DEAD_T = color(130, 138, 165)
V.toast_push = function() end V.on_kill_fx = function() end V.E_LIST = {} V.E_N = _v(0) V.last_shooting = _v(false) V.last_state = _v("standing") _G.__rt_reset_stats = function()
for k in pairs(V.stats) do V.stats[k] = 0 end end V.tick_counter = _v(0) V.cur_tgt = _v(nil) V.is_scoped_now = _v(false) V.now_tick = function() local ok, t = pcall(globals.tickcount)
if ok and type(t) == "number" then return t end return V.tick_counter end V.push = function(h, v, max) local n = h.n or 0 if n < max then n = n + 1 h[n] = v else for i = 1, max - 1 do h[i] = h[i + 1] end h[max] = v end
h.n = n return n end V.pct = function(a, total) if total <= 0 then return 0 end return math.floor((a * 100) / total + 0.5) end V.norm = function(a) a = a % 360 if a > 180 then a = a - 360 end return a end
V.vx_up = function(v) return v.X, v.Y, v.Z end V.vx_lo = function(v) return v.x, v.y, v.z end V.vxyz = function(v) if v == nil or v == false then return nil end local tv = type(v)
if tv == "number" or tv == "string" or tv == "boolean" then return nil end local ok, x, y, z = pcall(V.vx_up, v) if ok and type(x) == "number" then return x, y, z end local ok2, x2, y2, z2 = pcall(V.vx_lo, v)
if ok2 and type(x2) == "number" then return x2, y2, z2 end return nil end V.dist3 = function(ax, ay, az, bx, by, bz) if not ax or not bx then return nil end
local dx, dy, dz = ax - bx, (ay or 0) - (by or 0), (az or 0) - (bz or 0) return math.sqrt(((dx * dx) + (dy * dy)) + (dz * dz)) end V.my_px = _v(nil) V.my_py = _v(nil) V.my_pz = _v(nil) V.pitch_up = _v(false)
V.manual_inv = _v(false) V.next_flick = _v(0) V.flick_until = _v(0) V.flick_sign = _v(1) V.flick_pitch = _v(89) V.shift_ready_at = _v(0) V.jit_side = _v(1) V.way_idx = _v(1) V.cur_mode = _v("Jitter") V.cur_offset = _v(0)
V.cur_inv = _v(false) V.brute_phase = _v(0) V.brute_inv = _v(false) V.brute_mult = _v(1.0) V.brute_add = _v(0) V.last_health = _v(nil) V.was_alive = _v(false) V.MODES = { "Jitter", "3-Way", "Random", "Sway" } V.DATA = {}
V.spin_until = _v(0) V.spin_flip = _v(false) V.STAGES = { { 0, 1.0, false, 86 }, { 14, 0.7, true, 72 }, { -14, 1.25, false, 86 }, { 0, 0.5, true, 58 } } V.brute_stage = _v(1) V.brute_stage_t = _v(0) V.brute_lim = _v(86)
V.apply_stage = function(i) local st = V.STAGES[i] V.brute_stage = i V.brute_add = st[1] V.brute_mult = st[2] V.brute_inv = st[3] V.brute_lim = st[4] V.brute_stage_t = V.now_s() end V.brute_shift = function()
if not V.anti_brute:get() then return end if V.brute_mode:get() == "Stages" then V.apply_stage((V.brute_stage % 4) + 1) V.next_flick = V.now_tick() + 1
pcall(V.toast_push, "STAGE " .. tostring(V.brute_stage), V.C_DEAD_T, "anti-brute switched angles") return end V.brute_lim = 86 V.brute_phase = V.brute_phase + 1 V.brute_inv = not V.brute_inv
V.brute_mult = 0.6 + (math.random(0, 1000) / 1000) * 0.8 V.brute_add = math.random(-18, 18) V.jit_side = -V.jit_side V.way_idx = math.random(1, 3) V.next_flick = V.now_tick() + 1 end V.on_got_hit = function(dmg)
V.stats.taken = V.stats.taken + 1 if dmg then V.toast_push("-" .. tostring(math.floor(dmg)), color(255, 160, 60), "hurt by enemy") if dmg >= 90 then V.stats.taken_hs = V.stats.taken_hs + 1 end else
V.toast_push("HURT", color(255, 160, 60), "hit by enemy") end V.brute_shift() end V.aa_offset = function(t, range, mode) if mode == "Mixed" then mode = V.cur_mode end if mode == "Jitter" then V.jit_side = -V.jit_side
return V.jit_side * range elseif mode == "3-Way" then V.way_idx = (V.way_idx % 3) + 1 return ({ -range, 0, range })[V.way_idx] elseif mode == "Random" then return math.random(-math.floor(range), math.floor(range))
elseif mode == "Sway" then return math.sin(t * 0.35 + V.brute_phase) * range end return 0 end V.aa_inverter = function(m) if m == "Static" then return V.manual_inv ~= V.brute_inv end
if m == "Random" then return math.random(0, 1) == 1 end if math.random(1, 5) ~= 1 then V.cur_inv = not V.cur_inv end return V.cur_inv end V.aa_on:set_callback(function()
if not V.aa_on:get() then V.api("aa.reset", aa.reset) end end) V.HP_NAMES = { "entity.get_health", "entity.get_player_health", "info.health", "entity.get_local_state", "LocalPlayer attribute Health" } V.LPI = false
V.get_attr = function(inst, name) return inst:GetAttribute(name) end V.find_lp = function() return game:GetService("Players").LocalPlayer end V.lp_attr = function(name) if not V.LPI then local ok, p = pcall(V.find_lp)
V.LPI = (ok and p) or false if not V.LPI then return nil end end local ok2, v = pcall(V.get_attr, V.LPI, name) if ok2 then return v end return nil end V.hp_prev = {} V.hp_src = _v(nil) V.hp_poll = _v(0)
V.armor_last = _v(nil) V.armor_live = _v(false) V.hp_val = function(i, lp, info) if i == 5 then return tonumber(V.lp_attr("Health")) end if i == 3 then return tonumber(info.health) end if i == 4 then
if type(entity.get_local_state) ~= "function" then return nil end local ok, r = pcall(entity.get_local_state) if ok and type(r) == "table" then return tonumber(r.health) or tonumber(r.hp) or tonumber(r.Health) end
return nil end local f = (i == 1) and entity.get_health or entity.get_player_health if type(f) ~= "function" then return nil end local ok, r = pcall(f, lp) if ok then return tonumber(r) end return nil end
V.read_hp = function(lp, info) if V.hp_src then return V.hp_val(V.hp_src, lp, info) end local av = V.hp_val(5, lp, info) if type(av) == "number" then V.hp_src = 5
V.log(120, 200, 255, "[NP] health source: " .. V.HP_NAMES[5]) return av end V.hp_poll = V.hp_poll + 1 if (V.hp_poll % 4) ~= 0 then return nil end for i = 1, _v(4) do local v = V.hp_val(i, lp, info)
if type(v) == "number" then if V.hp_prev[i] and V.hp_prev[i] ~= v then V.hp_src = i V.log(120, 200, 255, "[NP] health source: " .. V.HP_NAMES[i]) return v end V.hp_prev[i] = v end end return nil end
V.read_armor = function(lp, info) local f = entity.get_player_armor or entity.get_armor if type(f) == "function" then local ok, r = pcall(f, lp) if ok and type(r) == "number" then return r end end
return tonumber(info.armor) end V.life_had = _v(false) V.life_gone_t = _v(nil) V.life_pend = _v(nil) V.life_last_ev = _v(-100) V.life_px = _v(nil) V.life_py = _v(nil) V.life_pz = _v(nil) V.local_state_logged = _v(false)
V.life_event = function(t) if (t - V.life_last_ev) > 3 then V.life_last_ev = t V.life_pend = t end end V.life_eval = function(now) if not V.life_pend or (now - V.life_pend) < 1.5 then return end local at = V.life_pend
V.life_pend = nil if V.hp_src then return end local seen, moved = 0, 0 for _, s in pairs(V.DATA) do if s.seen_t and (now - s.seen_t) < 10 then seen = seen + 1
if s.tp_t and math.abs(s.tp_t - at) < 2.5 then moved = moved + 1 end end end if seen >= 2 and moved >= math.ceil(seen * 0.5) then
V.log(120, 200, 255, "[NP] round restart detected (" .. tostring(moved) .. "/" .. tostring(seen) .. " enemies respawned)") return end V.stats.deaths = V.stats.deaths + 1 V.stats.streak = 0
V.toast_push("DEAD", V.C_DEAD_T, "you died  ·  death " .. tostring(V.stats.deaths)) V.log(255, 120, 120, "[NP] DEATH detected (respawn) | deaths " .. tostring(V.stats.deaths)) V.brute_shift() end
V.log_local_state = function() V.local_state_logged = true if type(entity.get_local_state) ~= "function" then return end local ok, r = pcall(entity.get_local_state) if not ok then return end if type(r) ~= "table" then
V.log(120, 200, 255, "[NP] local_state = " .. tostring(r)) return end local out, n = "", 0 for k, v in pairs(r) do if n < 24 then n = n + 1; out = out .. tostring(k) .. "=" .. tostring(v) .. " " end end
V.log(120, 200, 255, "[NP] local_state: " .. out) end V.peek_until = _v(0) V.lim_next = _v(0) V.lim_swap = _v(false) V.lim_rl = _v(1) V.lim_rr = _v(1) V.lim_shown_l = _v(86) V.lim_shown_r = _v(86) V.hold_until = _v(0)
V.rc = { params = false, char_t = -100, char = false, warned = false } V.v3 = function(p) local x, y, z = V.vxyz(p) if not x then return false end local ok, v = pcall(Vector3.new, x, y, z) return ok and v or false end
V.rc_update = function(now) if (now - V.rc.char_t) < 2 and V.rc.params then return end V.rc.char_t = now pcall(function() local ch = game:GetService("Players").LocalPlayer.Character local prm = RaycastParams.new()
prm.FilterType = Enum.RaycastFilterType.Exclude prm.FilterDescendantsInstances = { ch } V.rc.params = prm V.rc.char = ch or false end) end V.can_see = function(a, b, ename) local dir = b - a
local res = workspace:Raycast(a, dir, V.rc.params) if res == nil then return true end local mdl = res.Instance:FindFirstAncestorOfClass("Model") return (mdl ~= nil) and (mdl.Name == ename) end
V.peek_tick = function(t, lp, my_speed) local now = V.now_s() V.rc_update(now) if not V.rc.params then return end local okh, hp0 = pcall(entity.get_player_hitbox_position, lp, "head")
local me = okh and V.v3(hp0) or false if not me then return end for i = 1, V.E_N do local e = V.E_LIST[i] local s = V.st(e) if s and s.alive and ((not s.dist) or s.dist < 450) then
local oke, ep = pcall(entity.get_player_hitbox_position, e, "head") local eh = oke and V.v3(ep) or false if eh then local okv, vis = pcall(V.can_see, me, eh, tostring(s.name)) if not okv then
if not V.rc.warned then V.rc.warned = true; V.report("raycast", vis) end else if vis and not s.vis then local oks, es = pcall(movement.get_speed, e) local espd = (oks and type(es) == "number") and es or 0
V.peek_until = t + V.def_peek_len:get() V.next_flick = t s.peek_t = now s.peek_by = (my_speed > espd) and "you" or "enemy" end s.vis = vis end end end end end V.KS = {
EN = { "1", "sit", "nice aa", "refund", "neverpeek > you", "go next", "u good?", "too easy", "uninstall", "bot?" }, EN_HS = { "tap", "1 tap", "head", "hs" },
RU = { "1", "сел", "куда пошел", "минус", "neverpeek > тебя", "бб", "изи", "слабо", "не повезло", "иди учись" }, RU_HS = { "тапнул", "в голову", "хс", "1 тап" }, } V.ks_last = _v(-100)
V.CHAT = { box = false, unstick = 0, hist = {}, last_idx = 0, api_warned = false } V.CHAT_SKIP = { "SynUI_Window", "SynLuaEditorGui", "AdminPanelCore" } V.chat_box = function() local cb = V.CHAT.box if cb then
local okp, par = pcall(function() return cb.Parent end) if okp and par then return cb end end local ok, box = pcall(function() return game:GetService("Players").LocalPlayer.PlayerGui.KontraHUD.Chat.InputFrame.TextBox
end) if ok and box then V.CHAT.box = box return box end local okf, found = pcall(function() local pg = game:GetService("Players").LocalPlayer:FindFirstChildOfClass("PlayerGui") for _, o in ipairs(pg:GetDescendants()) do
if o:IsA("TextBox") then local skip = false for k = 1, #V.CHAT_SKIP do if o:FindFirstAncestor(V.CHAT_SKIP[k]) then skip = true end end local nm = string.lower(o:GetFullName())
if not skip and (string.find(nm, "chat", 1, true) or string.find(nm, "msg", 1, true) or string.find(nm, "message", 1, true)) then return o end end end return false end) V.CHAT.box = (okf and found) or false
return V.CHAT.box end V.CHAT_TAGS = { ".", "-", ",", "~", "..", "--", ".-", "-.", "~.", ". -", "- .", "~ -" } V.chat_dedupe = function(msg) local now = V.now_s() local h = V.CHAT.hist[msg]
if not h or (now - h.t) > 60 then V.CHAT.hist[msg] = { t = now, n = 0 } return msg end h.t = now h.n = h.n + 1 local tag = V.CHAT_TAGS[((h.n - 1) % #V.CHAT_TAGS) + 1] return msg .. " " .. tag end
V.chat_unstick = function() pcall(function() game:GetService("GuiService").SelectedObject = nil end) pcall(function() local uis = game:GetService("UserInputService") local f = uis:GetFocusedTextBox()
if f then f:ReleaseFocus(false) end if not V.ui_open then uis.MouseBehavior = Enum.MouseBehavior.LockCenter end end) if not V.ui_open then pcall(input.set_cursor_unlocked, false) end end
client.set_event_callback("paint", function() if V.CHAT.unstick > 0 then V.CHAT.unstick = V.CHAT.unstick - 1 V.chat_unstick() end end) V.chat_api = function(final)
if type(chat) ~= "table" or type(chat.say) ~= "function" then return false end local ok, sent, err = pcall(chat.say, final) if ok and sent ~= false then return true end if not V.CHAT.api_warned then
V.CHAT.api_warned = true V.log(255, 170, 90, "[NP] chat.say failed (" .. tostring(ok and err or sent) .. "), using chat box fallback") end return false end V.chat_send = function(msg) local final = V.chat_dedupe(msg)
local okapi, sent = pcall(V.chat_api, final) if okapi and sent then return true end local ok_t, typing = pcall(input.is_typing) if ok_t and typing == true then return false end local box = V.chat_box() if not box then
V.report("chat", "chat textbox not found (KontraHUD.Chat.InputFrame.TextBox)") return false end local ok, err = pcall(function() box:CaptureFocus() box.Text = final box:ReleaseFocus(true) end) V.CHAT.unstick = 6
V.chat_unstick() if not ok then V.report("chat send", err) end return ok end V.killsay = function(headshot) if not V.ks_on:get() then return end local now = V.now_s() if (now - V.ks_last) < V.ks_cd:get() then return end
local mode = V.ks_mode:get() if mode == "Headshots" and not headshot then return end local st = V.stats.streak if mode == "Streaks" and not (st == 3 or st == 5 or st == 7 or st >= 10) then return end
local lang = V.ks_lang:get() local pool = V.KS[lang] if headshot then pool = V.KS[lang .. "_HS"] end local idx = math.random(1, #pool) if idx == V.CHAT.last_idx and #pool > 1 then idx = (idx % #pool) + 1 end
V.CHAT.last_idx = idx local msg = pool[idx] if mode == "Streaks" then msg = msg .. "  [" .. tostring(st) .. " streak]" end if V.chat_send(msg) then V.ks_last = now end end V.ks_test:set_callback(function()
if V.chat_send("neverpeek test") then V.log(150, 255, 150, "[NP] chat: test message sent (check the game chat)") end end) V.aa_tick = function(t) local now = V.now_s() V.life_eval(now)
local ok, lp = V.api("entity.get_local_player", entity.get_local_player) local info = (ok and lp) and select(2, V.api("entity.get_info", entity.get_info, lp)) or nil if not info then V.last_health = nil
if V.life_had and not V.life_gone_t then V.life_gone_t = now end return end if V.life_gone_t and (now - V.life_gone_t) > 0.25 then V.life_event(V.life_gone_t) end V.life_gone_t = nil V.life_had = true
if not V.local_state_logged then V.log_local_state() end local pos = nn(info.position, false) if not pos and type(entity.get_local_position) == "function" then local okp, pv = pcall(entity.get_local_position)
pos = okp and pv or false end local px, py, pz = V.vxyz(pos or nil) if px then if V.life_px then local d = V.dist3(px, py, pz, V.life_px, V.life_py, V.life_pz) if d and d > 30 then V.life_event(now) end end
V.life_px = px V.life_py = py V.life_pz = pz V.my_px = px V.my_py = py V.my_pz = pz end local hp = V.read_hp(lp, info) local alive = true if entity.is_alive then
local oka, a = V.api("entity.is_alive", entity.is_alive, lp) if oka and a ~= nil then alive = a end elseif info.is_alive == false then alive = false end if hp ~= nil and hp <= 0 then alive = false end
if V.lp_attr("IsDead") == true then alive = false end if V.was_alive and not alive then V.stats.deaths = V.stats.deaths + 1 V.stats.streak = 0 V.life_last_ev = now V.brute_shift() end V.was_alive = alive
if not alive then V.last_health = nil return end if type(hp) == "number" then if V.last_health and hp < V.last_health then V.on_got_hit(V.last_health - hp) end V.last_health = hp elseif not V.hp_src then
local ar = V.read_armor(lp, info) if type(ar) == "number" then if V.armor_last and ar < V.armor_last then V.armor_live = true V.on_got_hit(nil) end V.armor_last = ar end end
local in_air = nn(info.is_in_air, info.state == "Jumping") local speed = info.speed or 0 local crouch = (info.is_crouching == true) or (info.state == "Duck") V.lp_speed = speed V.lp_air = in_air and true or false
if in_air and crouch then V.last_state = "air crouch" elseif in_air then V.last_state = "in air" elseif crouch then V.last_state = "crouching" elseif speed > 5 then V.last_state = "moving" else V.last_state = "standing"
end if not V.aa_on:get() then return end local P_mode, P_range, P_add, P_lim = V.yaw_mode:get(), V.yaw_range:get(), V.yaw_add:get(), V.desync_lim:get() local P_limR = V.desync_lim_r:get()
local P_inv, P_def = V.inv_mode:get(), true if V.sa_on:get() then local c = V.SA_BY[V.last_state] if c and c.ov:get() then P_mode = c.mode:get() P_range = c.range:get() P_add = c.add:get() P_lim = c.lim:get()
P_limR = c.rlim:get() P_inv = c.inv:get() P_def = c.def:get() end end local def_active = V.def_on:get() and P_def local trig = V.def_trig:get() if def_active and trig ~= "Timer" and (t % 6) == 0 then
local okpk, epk = pcall(V.peek_tick, t, lp, speed) if not okpk then V.report("peek", epk) end end local in_peek = t < V.peek_until if V.brute_mode:get() == "Stages" and V.brute_stage > 1 and V.brute_reset:get() > 0
and (now - V.brute_stage_t) > V.brute_reset:get() then V.apply_stage(1) end local shooting = false if V.acc_shoot:get() then if V.cur_tgt then V.shot_win_t = now end
shooting = (V.cur_tgt ~= nil) or ((now - V.shot_win_t) < 0.6) or ((now - V.DT.shot_t) < 0.5) or in_peek end V.last_shooting = shooting local timer_ok = (trig ~= "On Peek") or in_peek
if def_active and timer_ok and not shooting and t >= V.next_flick and t >= V.flick_until then V.flick_until = t + V.def_len:get() if in_peek then V.next_flick = t + V.def_len:get() + math.random(1, 3) else
V.next_flick = t + V.def_interval:get() + math.random(-2, 3) end V.flick_sign = (math.random(0, 1) == 1) and 1 or -1 V.flick_pitch = (math.random(1, 4) == 1) and 0 or 89
if V.def_shift:get() and t >= V.shift_ready_at and not V.smart_hold() then local _, charged = V.api("rage.is_double_tap_charged", rage.is_double_tap_charged) if charged then V.api("rage.shift_ticks", rage.shift_ticks)
V.shift_ready_at = t + 24 + math.random(0, 12) end end if V.def_air_lc:get() and in_air and not V.smart_hold() then V.api("rage.shift_ticks", rage.shift_ticks, 1) end end
local flicking = def_active and t < V.flick_until and not shooting local _, choking = V.api("aa.is_fake_lag_choking", aa.is_fake_lag_choking) if ((not choking) or (not V.lag_on:get())) and t >= V.hold_until then
local hold = V.mod_hold:get() V.hold_until = t + hold if V.mod_hold_rnd:get() and hold > 1 then V.hold_until = t + math.random(1, hold) end if P_mode == "Mixed" and math.random(1, 6) == 1 then
V.cur_mode = V.MODES[math.random(1, #V.MODES)] end V.cur_offset = V.aa_offset(t, P_range * V.brute_mult, P_mode) V.cur_inv = V.aa_inverter(P_inv) end
if shooting and V.aa_shot:get() == "Disable" and V.spin_until <= now then V.aa_released = true V.api("aa.override_pitch", aa.override_pitch, "Disabled") V.api("aa.override_yaw", aa.override_yaw, 0, "camera")
V.api("aa.override_yaw_modifier", aa.override_yaw_modifier, "Disabled") V.api("aa.override_yaw_offset", aa.override_yaw_offset, 0) V.api("aa.override_body_yaw", aa.override_body_yaw, false) if V.lag_on:get() then
V.api("aa.override_fake_lag_limit", aa.override_fake_lag_limit, V.acc_lag:get()) V.api("aa.override_fake_lag_variability", aa.override_fake_lag_variability, 0) else
V.api("aa.override_fake_lag", aa.override_fake_lag, false) end return end V.aa_released = false if V.spin_until > now then V.spin_flip = not V.spin_flip
V.api("aa.override_pitch", aa.override_pitch, V.spin_flip and 89 or -89) elseif flicking then V.api("aa.override_pitch", aa.override_pitch, V.flick_pitch) else
V.api("aa.override_pitch", aa.override_pitch, V.pitch_up and "Up" or "Down") end V.api("aa.override_yaw", aa.override_yaw, 180, "camera") V.api("aa.override_yaw_modifier", aa.override_yaw_modifier, "Disabled")
if flicking then V.api("aa.override_yaw_offset", aa.override_yaw_offset, V.flick_sign * math.random(120, 180)) else V.api("aa.override_yaw_offset", aa.override_yaw_offset, V.norm(V.cur_offset + P_add + V.brute_add)) end
local limL = P_lim local limR = P_limR local lj = V.lim_jit:get() if lj ~= "Off" and t >= V.lim_next then V.lim_next = t + V.lim_jit_spd:get() V.lim_swap = not V.lim_swap V.lim_rl = math.random(40, 100) / 100
V.lim_rr = math.random(40, 100) / 100 end if lj == "Swap" and V.lim_swap then limL = P_limR limR = P_lim elseif lj == "Random" then limL = math.floor((P_lim * V.lim_rl) + 0.5) limR = math.floor((P_limR * V.lim_rr) + 0.5)
end if V.hf_on:get() then local iv = V.hf_ms:get() if V.hf_speed:get() == "Wave" then local wv = (math.sin(now * (V.hf_cycle:get() * 0.5)) + 1) / 2 iv = iv + ((V.hf_ms2:get() - iv) * wv) end
if (now - V.hf_last) >= (iv / 1000) then V.hf_last = now V.hf_state = not V.hf_state if V.hf_state then V.hf_cur = V.hf_rnd_a:get() and math.random(0, 86) or V.hf_a:get() else
V.hf_cur = V.hf_rnd_b:get() and math.random(0, 86) or V.hf_b:get() end end limL = V.hf_cur limR = V.hf_cur end if V.anti_brute:get() and V.brute_mode:get() == "Stages" then limL = math.min(limL, V.brute_lim)
limR = math.min(limR, V.brute_lim) end V.lim_shown_l = limL V.lim_shown_r = limR V.api("aa.override_body_yaw", aa.override_body_yaw, true) V.api("aa.override_desync_limits", aa.override_desync_limits, limL, limR)
V.api("aa.override_inverter", aa.override_inverter, V.cur_inv ~= V.brute_inv) if V.lag_on:get() then local limit, var = V.lag_limit:get(), V.lag_var:get()
if V.lag_adapt:get() and (in_air or speed > 5) then limit = 14; var = math.max(var, 60) end if shooting then limit = V.acc_lag:get(); var = 0 end V.api("aa.override_fake_lag", aa.override_fake_lag, true)
V.api("aa.override_fake_lag_limit", aa.override_fake_lag_limit, limit) V.api("aa.override_fake_lag_variability", aa.override_fake_lag_variability, var) else V.api("aa.override_fake_lag", aa.override_fake_lag, false) end
end V.KEYC = {} V.key_of = function(p) if p == nil then return nil end local c = V.KEYC[p] if c then return c end local ok, info = pcall(entity.get_info, p) if ok and info and info.name then V.KEYC[p] = info.name
return info.name end return tostring(p) end V.st = function(p) local k = V.key_of(p) if not k then return nil end if not V.DATA[k] then V.DATA[k] = { misses = 0, stage = 1, hit_off = nil, yaw_hist = {}, jitter = false,
side = 1, offset = 0, shot_off = nil, spread_miss = 0, total_res = 0, hard = false, pitch_hist = {}, pitch_forced = false, defensive = false, pdef = false, pbest = 0,
cls = nil, cls_cand = nil, cls_cnt = 0, hits_n = 0, miss_n = 0, dead_n = 0, res_yaw = nil, dist = nil, alive = true, name = k, ascore = {}, soft = 0 } end return V.DATA[k] end do local base_reset = _G.__rt_reset_stats
_G.__rt_reset_stats = function() if base_reset then base_reset() end for _, d in pairs(V.DATA) do d.hits_n, d.miss_n, d.dead_n = 0, 0, 0 end end end
V.CLS_SHORT = { STATIC = "static", JITTER = "jitter", SPIN = "spin", DEFENSIVE = "def", RANDOM = "rand" } V.cls_label = function(s) if s and s.noaa then return "no aa" end if not s or not s.cls then return nil end
local b = V.CLS_SHORT[s.cls] or "?" if s.pdef and s.cls ~= "DEFENSIVE" then b = b .. "+def" end return b end V.update_history = function(s, yaw) if type(yaw) ~= "number" then return end local h = s.yaw_hist
local hn = h.n or 0 if hn > 0 and math.abs(V.norm(yaw - h[hn])) < 0.5 then s.frz = (s.frz or 0) + 1 if s.frz < 16 then return end end s.frz = 0 local n = V.push(h, yaw, 16) if n < 6 then s.jitter = false return end
local flips = 0 local big = 0 local prev = false local from = math.max(2, n - 10) for i = from, n do local d = V.norm(h[i] - h[i - 1]) if math.abs(d) > 25 then big = big + 1 end
if prev and d * prev < 0 and math.abs(d) > 15 then flips = flips + 1 end prev = d end s.jitter = (flips >= 3 and big >= 4) end V.classify = function(s) local h = s.yaw_hist local n = h.n or 0 if n < 8 then return nil end
local flips = 0 local big = 0 local small = 0 local same = 0 local spikes = 0 local sum = 0 local m = 0 local prev = false for i = 2, n do local d = V.norm(h[i] - h[i - 1]) local ad = math.abs(d) if ad > 90 then
spikes = spikes + 1 prev = false else m = m + 1 if ad > 25 then big = big + 1 end if ad < 8 then small = small + 1 end if prev then local ap = math.abs(prev)
if d * prev < 0 and ad > 15 and ap > 15 then flips = flips + 1 end if d * prev > 0 and ad > 12 and ap > 12 then same = same + 1 end end sum = sum + d prev = d end end local now = V.now_s()
if spikes >= 2 then s.spike_t = now end if m < 4 then if s.spike_t and (now - s.spike_t) < 3 then return "DEFENSIVE" end return nil end if same >= ((m - 1) * 0.7) and math.abs(sum) > 120 then return "SPIN" end
if flips >= (m * 0.5) then return "JITTER" end if s.spike_t and (now - s.spike_t) < 3 then return "DEFENSIVE" end if flips >= 3 and big >= 4 then return "JITTER" end if small >= (m * 0.75) then return "STATIC" end
return "RANDOM" end V.update_class = function(s) local c = V.classify(s) if not c then return end if c == s.cls_cand then s.cls_cnt = s.cls_cnt + 1 else s.cls_cand, s.cls_cnt = c, 1 end
if s.cls_cnt >= 3 then s.cls = c end end V.pitch_bucket = function(p) if p < -45 then return -89 elseif p > 45 then return 89 end return 0 end V.sample_pitch = function(s, enemy, info) local p = false
if entity.get_player_angles_deg then local ok, pd = V.api("entity.get_player_angles_deg", entity.get_player_angles_deg, enemy) if ok and type(pd) == "number" then p = pd end end
if not p and type(info.pitch) == "number" then local ip = info.pitch p = (math.abs(ip) <= 1.6) and math.deg(ip) or ip end if type(p) ~= "number" then return end local h = s.pitch_hist local n = V.push(h, p, 16)
local mn, mx = math.huge, -math.huge local c_dn, c_z, c_up = 0, 0, 0 for i = 1, n do local v = h[i] if v < mn then mn = v end if v > mx then mx = v end local b = V.pitch_bucket(v)
if b < 0 then c_dn = c_dn + 1 elseif b > 0 then c_up = c_up + 1 else c_z = c_z + 1 end end s.pdef = (n >= 8 and (mx - mn) > 60) local best, bc = 0, c_z if c_dn > bc then best = -89; bc = c_dn end
if c_up > bc then best = 89; bc = c_up end s.pbest = best end V.resolve_pitch = function(s, enemy, pset_fn) local mode = V.enemy_pitch:get() s.defensive = s.pdef and true or false if mode == "Force 0" then
pset_fn(enemy, "Force pitch", true) pset_fn(enemy, "Force pitch value", 0) s.pitch_forced = true return end if mode == "Auto" and s.pdef then pset_fn(enemy, "Force pitch", true)
pset_fn(enemy, "Force pitch value", s.pbest) s.pitch_forced = true elseif s.pitch_forced then pset_fn(enemy, "Force pitch", false) s.pitch_forced = false end end V.build_seq = function(s)
local m, side = V.max_desync:get(), s.side local c = V.res_ptype:get() and s.cls or nil local h = m * 0.5 if c == "JITTER" or (c == nil and s.jitter) then return { 0, side * h, -side * h, side * m, -side * m } end
if c == "SPIN" then return { 0, side * h, -side * h } end if c == "RANDOM" then return { 0, side * m, -side * m } end if c == "DEFENSIVE" then return { side * m, -side * m, 0 } end
return { side * m, -side * m, side * h, -side * h, 0 } end V.ak = function(a) return tostring(math.floor((tonumber(a) or 0) + 0.5)) end V.ascore_add = function(s, a, d) if not s or a == nil then return end
if not s.ascore then s.ascore = {} end local k = V.ak(a) s.ascore[k] = math.max(-15, math.min(25, (s.ascore[k] or 0) + d)) end V.sc_cmp = function(a, b) if a.sc ~= b.sc then return a.sc > b.sc end return a.i < b.i end
V.ANIM = { sign = 1, good = 0, bad = 0 } V.anim_angle = function(anim) local off = false local gf = tonumber(anim.goal_feet_yaw) local ey = tonumber(anim.eye_angles_y) if gf and ey then off = V.norm(gf - ey) end
if (not off or math.abs(off) < 4) and type(anim.desync) == "number" then off = anim.desync end if not off or math.abs(off) < 4 then return false end local m = V.max_desync:get()
off = math.max(-m, math.min(m, off)) * V.ANIM.sign return math.floor(off + 0.5) end V.anim_usable = function(s, now) if not V.res_anim:get() or not s.anim_off then return false end if s.noaa then return false end
if s.cls == "JITTER" or s.cls == "SPIN" or s.cls == "RANDOM" then return false end if now < (s.anim_ban or 0) then return false end return true end V.anim_result = function(s, good) if not s.shot_anim then return end
if good then V.ANIM.good = V.ANIM.good + 1 else V.ANIM.bad = V.ANIM.bad + 1 s.anim_ban = V.now_s() + 4 end if V.ANIM.bad >= 3 and V.ANIM.bad >= ((V.ANIM.good * 2) + 3) then V.ANIM.sign = -V.ANIM.sign V.ANIM.good = 0
V.ANIM.bad = 0 V.log(120, 200, 255, "[NP] anim resolver: flipped side convention") end end V.dsync_seen = false V.noaa_update = function(s, desync) if type(desync) ~= "number" then s.noaa = false return end
local ad = math.abs(desync) if ad > 5 then V.dsync_seen = true s.lowd = 0 if s.noaa then s.noaa = false end return end s.lowd = math.min(64, (s.lowd or 0) + 1)
s.noaa = V.res_noaa:get() and V.dsync_seen == true and s.lowd >= 24 and s.cls ~= "SPIN" and s.cls ~= "JITTER" end V.pick_offset = function(s) if s.noaa then return 0 end
if V.remember_hit:get() and s.hit_off ~= nil and s.misses == 0 then return s.hit_off end local seq = V.build_seq(s) if V.anim_usable(s, V.now_s()) then local dup = false for _, v in ipairs(seq) do
if math.abs(v - s.anim_off) < 6 then dup = true end end if not dup then table.insert(seq, 1, s.anim_off) end end if V.res_score:get() and s.ascore then local arr = {}
for i, v in ipairs(seq) do arr[i] = { v = v, i = i, sc = s.ascore[V.ak(v)] or 0 } end table.sort(arr, V.sc_cmp) for i, it in ipairs(arr) do seq[i] = it.v end end return seq[((s.stage - 1) % #seq) + 1] end
V.want_baim = function(s) if not s then return false end local ba = V.baim_after:get() if ba > 0 and s.misses >= ba then return true end if V.hard_on:get() and s.hard then return true end
if V.res_ptype:get() and (s.cls == "SPIN" or s.cls == "RANDOM") and s.misses >= 1 then return true end return false end V.PL = {} V.pset = function(enemy, k, v) if not V.PL[enemy] then V.PL[enemy] = {} end
local t = V.PL[enemy] if t[k] == v then return true end local ok = V.api("plist.set", plist and plist.set, enemy, k, v) if ok then t[k] = v end return ok end V.builtin_was = _v(nil) V.lua_works = _v(false)
V.restore_builtin = function() if V.builtin_was ~= nil then V.api("rage.set_resolver_enabled", rage.set_resolver_enabled, V.builtin_was) V.builtin_was = nil end end V.anim_broken = _v(false) V.twist_broken = _v(false)
V.panim_broken = _v(false) V.dsync_broken = _v(false) V.anim_logged = _v(false) V.BI_MODES = { "Brute-force", "Adaptive", "Freestanding" } V.bi_idx = _v(1) V.bi_last = _v(0) V.bi_synced = _v(false) V.bi_set = function(i)
V.bi_idx = i V.api("rage.set_resolver_mode", rage.set_resolver_mode, V.BI_MODES[V.bi_idx]) end V.bi_next_mode = function() V.bi_set((V.bi_idx % 3) + 1) end V.bi_for = function(c) if c == "STATIC" then return 3 end
if c == nil then return nil end return 1 end V.bi_cycle_tick = function() if V.res_mode:get() == "Lua Brute" then return end local m = V.bi_cycle:get() if m == "Per AA Type" then local tgt = V.cur_tgt
if tgt and tgt.player then local s = V.st(tgt.player) local want = s and V.bi_for(s.cls) if want and (want ~= V.bi_idx or not V.bi_synced) then V.bi_set(want); V.bi_synced = true end end return end V.bi_synced = false
if m ~= "Every Second" then return end local now = V.now_s() if now - V.bi_last >= 1 then V.bi_last = now V.bi_next_mode() end end V.resolver_tick = function() if not V.res_on:get() then if V.builtin_was ~= nil then
V.restore_builtin() V.api("plist.reset_all", plist and plist.reset_all) V.PL = {} end return end local ok, enemies = V.api("entity.get_enemies", entity.get_enemies) if not ok or type(enemies) ~= "table" then return end
local now = V.now_s() local want_cls = V.res_cls:get() local applied = false V.E_N = 0 for _, enemy in ipairs(enemies) do if V.E_N < 10 then V.E_N = V.E_N + 1; V.E_LIST[V.E_N] = enemy end local s = V.st(enemy) if s then
s.seen_t = now local _, info = V.api("entity.get_info", entity.get_info, enemy) if info then local ex, ey, ez = V.vxyz(info.position) if ex then if s.px then local jd = V.dist3(ex, ey, ez, s.px, s.py, s.pz)
if jd and jd > 30 then s.tp_t = now; s.hp = 100 end end s.px = ex s.py = ey s.pz = ez s.dist = V.dist3(ex, ey, ez, V.my_px, V.my_py, V.my_pz) end end if info and type(info.health) == "number" then
if s.hp ~= info.health then s.hp_t = now end s.hp = info.health end if info and info.is_alive ~= false then if not s.alive then s.tp_t = now end s.alive = true local yaw = nn(info.yaw, false)
local desync = nn(info.desync_delta, false) local anim = false if not V.anim_broken and entity.get_anim_state then local oka, a = V.api("entity.get_anim_state", entity.get_anim_state, enemy)
if not oka then V.anim_broken = true else anim = a end end if not anim and not V.panim_broken and entity.get_player_anim_state then
local oka, a = V.api("entity.get_player_anim_state", entity.get_player_anim_state, enemy) if not oka then V.panim_broken = true else anim = a end end s.anim_off = false if type(anim) == "table" then
local aoff = V.anim_angle(anim) if aoff then s.anim_off = aoff end yaw = anim.eye_angles_y or yaw desync = anim.desync or desync if not V.anim_logged then V.anim_logged = true
V.log(120, 200, 255, "[NP] anim state works: desync=" .. tostring(anim.desync) .. " eye=" .. tostring(anim.eye_angles_y)) end end
if (type(desync) ~= "number" or math.abs(desync) <= 5) and not V.dsync_broken and entity.get_desync_delta then local okd, dd = V.api("entity.get_desync_delta", entity.get_desync_delta, enemy)
if not okd then V.dsync_broken = true elseif type(dd) == "number" then desync = dd end end if type(desync) ~= "number" or math.abs(desync) <= 5 then if not V.twist_broken and entity.get_player_twist_deg then
local okt, tw_deg = V.api("entity.get_player_twist_deg", entity.get_player_twist_deg, enemy) if not okt then V.twist_broken = true elseif type(tw_deg) == "number" then desync = tw_deg end end end
if V.jitter_det:get() or want_cls then V.update_history(s, yaw) end if want_cls or V.enemy_pitch:get() == "Auto" then V.sample_pitch(s, enemy, info) end if want_cls then V.update_class(s) end
if s.misses == 0 and type(desync) == "number" and math.abs(desync) > 5 then s.side = desync > 0 and 1 or -1 end V.noaa_update(s, desync) if V.res_mode:get() == "Lua Brute" then s.offset = V.pick_offset(s)
local a1 = V.pset(enemy, "Force body yaw", true) local a2 = V.pset(enemy, "Override yaw offset", s.offset) if a1 and a2 then applied = true end s.yaw_forced = true else s.offset = 0 if V.anim_usable(s, now) then
s.anim_on = true s.offset = s.anim_off local a1 = V.pset(enemy, "Force body yaw", true) V.pset(enemy, "Override yaw offset", s.offset) if a1 then s.yaw_forced = true end else s.anim_on = false if s.yaw_forced then
V.pset(enemy, "Force body yaw", false) s.yaw_forced = false end end end local sp = V.sp_after:get() local force_baim = V.want_baim(s) local hard_force = force_baim and (V.baim_mode:get() == "Force")
V.pset(enemy, "Override prefer body aim", hard_force and "Force" or "-") V.pset(enemy, "Force body aim", hard_force and true or false) V.pset(enemy, "Override safe point", (sp > 0 and s.misses >= sp) and "Force" or "-")
V.resolve_pitch(s, enemy, V.pset) elseif info and info.is_alive == false then s.alive = false s.yaw_hist = {} s.pitch_hist = {} s.defensive = false s.pdef = false s.cls_cand = nil s.cls_cnt = 0 end end end
if applied then V.lua_works = true end if V.res_mode:get() ~= "Lua Brute" then V.lua_works = false if V.builtin_was ~= nil then V.restore_builtin() V.log(0, 200, 255, "[NP] Built-in resolver restored (Assist mode)") end
elseif V.kill_builtin:get() and V.lua_works and V.builtin_was == nil then local ok2, v = V.api("rage.is_resolver_enabled", rage.is_resolver_enabled) V.builtin_was = (ok2 and v ~= nil) and v or true
V.api("rage.set_resolver_enabled", rage.set_resolver_enabled, false) V.log(0, 200, 255, "[NP] Lua resolver active, built-in disabled") elseif (not V.kill_builtin:get()) and V.builtin_was ~= nil then V.restore_builtin()
end end V.hc_applied = _v(nil) V.baim_applied = _v(false) V.prio_applied = _v(nil) V.sp_applied = _v(false) V.def_body_until = _v(0) V.baim_pref = _v(false) V.PRIO = { ["Body First"] = { "chest", "pelvis", "head" },
["Head First"] = { "head", "chest", "pelvis" }, } V.rage_reset = function() V.api("rage.reset", rage.reset) V.baim_applied = false V.hc_applied = nil V.prio_applied = nil V.sp_applied = false V.md_applied = false end
V.md_applied = false V.lp_speed = 0 V.lp_air = false V.SNIPER_KEYS = { "ssg", "sniper" } V.scope_blocked = _v(false) V.scoped_since = _v(nil) V.is_sniper = function(w)
if w.is_sniper ~= nil then return w.is_sniper == true end local str = string.lower(tostring(w.name or "") .. " " .. tostring(w.type or "") .. " " .. tostring(w.id or "")) for _, k in ipairs(V.SNIPER_KEYS) do
if string.find(str, k, 1, true) then return true end end return false end V.scope_tick = function() local block = false local okw, w = V.api("client.get_weapon", client.get_weapon)
V.is_scoped_now = okw and type(w) == "table" and w.is_scoped == true if okw and type(w) == "table" then V.wpn_name = string.lower(tostring(w.name or w.id or "")) end
if V.scope_wait:get() and okw and type(w) == "table" and V.is_sniper(w) then local scoped = (w.is_scoped == true) or ((tonumber(w.scope_level) or 0) > 0) if scoped then local now = V.now_s()
if not V.scoped_since then V.scoped_since = now end block = ((now - V.scoped_since) * 1000) < V.scope_delay:get() else V.scoped_since = nil end else V.scoped_since = nil end if block and not V.scope_blocked then
V.api("rage.override_hitchance", rage.override_hitchance, 100) V.scope_blocked = true elseif (not block) and V.scope_blocked then V.rage_reset() V.scope_blocked = false end end V.HD = {} V.plr_helmet = function(target)
local name = V.key_of(target) local plr = game:GetService("Players"):FindFirstChild(tostring(name)) if not plr then return "u" end local h = plr:GetAttribute("HasHelmet") if h == true then return "h" end
if h == false then return "n" end return "u" end V.soft_hit = function(s, hb, dmg, killed, target) if not V.res_dmgval:get() or killed or hb ~= "head" then return false end local w0 = V.wpn_name or ""
if w0 == "" then return false end local okh, hm = pcall(V.plr_helmet, target) local w = w0 .. ":" .. ((okh and hm) or "u") local ref = V.HD[w] or 0 if dmg > ref then V.HD[w] = dmg end
if ref <= 0 or dmg >= (ref * 0.6) then return false end s.soft = (s.soft or 0) + 1 V.ascore_add(s, s.shot_off or s.offset, -0.5) if V.log_shots:get() then
V.log(255, 200, 80, string.format("[NP] SOFT HIT %s | head dmg %d < %d | soft %d/2", tostring(V.key_of(target)), math.floor(dmg), math.floor(ref * 0.6), s.soft)) end if s.soft >= 2 then s.soft = 0 s.misses = s.misses + 1
s.stage = s.stage + 1 if s.hit_off ~= nil and s.shot_off == s.hit_off then s.hit_off = nil end end return true end V.BD = {} V.BD_FALLBACK = { { "ssg", 70 }, { "deagle", 40 }, { "zeu", 100 }, { "taser", 100 } }
V.BODY_PARTS = { "chest", "pelvis", "stomach", "torso", "body", "spine", "abdomen" } V.is_body_part = function(hb) for _, k in ipairs(V.BODY_PARTS) do if string.find(hb, k, 1, true) then return true end end return false
end V.body_learn = function(hb, dmg) if not V.is_body_part(hb) then return end local w = V.wpn_name or "" if w == "" then return end local old = V.BD[w]
if old then V.BD[w] = (old * 0.6) + (dmg * 0.4) else V.BD[w] = dmg end end V.body_est = function() local w = V.wpn_name or "" if V.BD[w] then return V.BD[w] end for _, p in ipairs(V.BD_FALLBACK) do
if string.find(w, p[1], 1, true) then return p[2] end end return 0 end V.is_lethal = function(tgt) local s = V.st(tgt.player) if not s then return false end local hp = tonumber(s.hp)
if not hp or hp <= 0 then return false end local td = tonumber(tgt.damage) local part = string.lower(tostring(tgt.part_name or "")) if td and td >= hp and V.is_body_part(part) then return "1 shot" end
local est = V.body_est() * 0.9 if est <= 0 then return false end if hp <= est then return "1 shot" end if V.smart_dt:get() and V.DT.on and V.DT.charge >= 14 and hp <= (est * 2) then return "dt" end return false end
V.lethal_now = false V.shot_win_t = -10 V.wpn_name = "" V.DT = { charge = 0, on = false, t = -10, shot_t = -10, tgt_t = -10, hold = "", ok = false } V.dt_read = function(now) if (now - V.DT.t) < 0.05 then return end
V.DT.t = now local c = false local ok1, r1 = pcall(rage.get_double_tap_charge) if ok1 and type(r1) == "number" then c = r1 end if c == false then local ok2, r2 = pcall(rage.get_dt_charge)
if ok2 and type(r2) == "number" then c = r2 end end if c == false then local ok3, r3 = pcall(rage.is_double_tap_charged) if ok3 and type(r3) == "boolean" then c = r3 and 14 or 0 end end V.DT.ok = c ~= false
V.DT.charge = math.max(0, math.min(14, (c ~= false) and c or 0)) local on = 0 local ok4, r4 = pcall(rage.is_double_tap_active) if ok4 and type(r4) == "boolean" then on = r4 end if on == 0 then
local ok5, r5 = pcall(rage.is_double_tap) if ok5 and type(r5) == "boolean" then on = r5 end end if on == 0 then on = V.DT.ok end V.DT.on = on if V.cur_tgt then V.DT.tgt_t = now end end V.smart_hold = function()
if not V.smart_dt:get() then V.DT.hold = "" return false end local now = V.now_s() V.dt_read(now) local why = "" if V.DT.charge < 14 then why = "charging" elseif V.cur_tgt or (now - V.DT.tgt_t) < 0.6 then why = "saving"
elseif (now - V.DT.shot_t) < 1 then why = "recharge" end V.DT.hold = why return why ~= "" end V.baim_tick = function() local want = false local tgt = V.cur_tgt if V.res_on:get() and tgt and tgt.player then
if V.want_baim(V.st(tgt.player)) then want = true end end local pref = want and (V.baim_mode:get() == "Prefer") if pref then want = false end V.lethal_now = false local lm = V.lethal_mode:get()
if lm ~= "Off" and tgt and tgt.player then local lk = V.is_lethal(tgt) if lk then V.lethal_now = lk if lm == "Force" then want = true else pref = true end end end if want then pref = false end V.baim_pref = pref
if want ~= V.baim_applied then V.api("rage.override_force_baim", rage.override_force_baim, want) V.baim_applied = want end end V.move_hc = function() local w = V.wpn_name or ""
if not (string.find(w, "ssg", 1, true) or string.find(w, "deagle", 1, true) or string.find(w, "desert", 1, true)) then return false end if V.lp_air then return V.mhc_air:get() end local sp = tonumber(V.lp_speed) or 0
if sp > 11 then return V.mhc_run:get() end if sp > 1.5 then return V.mhc_walk:get() end return false end V.md_guess = function() local best = false local bd = 260 local cx = V.sw / 2 local cy = V.sh / 2
for i = 1, V.E_N do local e = V.E_LIST[i] local s = e and V.st(e) if s and s.alive and s.px then local ok, x, y, on = pcall(render.world_to_screen, Vector3.new(s.px, s.py + 2, s.pz))
if ok and on and type(x) == "number" then local d = math.sqrt(((x - cx) * (x - cx)) + ((y - cy) * (y - cy))) if d < bd then bd = d best = e end end end end return best end V.md_tick = function() local want = false
if V.dyn_md:get() then local tgt = V.cur_tgt local tp = (tgt and tgt.player) or false if not tp then local okg, g = pcall(V.md_guess) if okg and g then tp = g end end if tp then local s = V.st(tp)
local hp = s and tonumber(s.hp) or 100 if hp > 0 then local w = V.wpn_name or "" local est = V.body_est() * 0.9 local dt_ok = V.smart_dt:get() and V.DT.on and V.DT.charge >= 14
if est > 0 and string.find(w, "ssg", 1, true) then if hp <= est then want = math.max(1, math.ceil(hp)) elseif dt_ok and hp <= (est * 2) then want = math.max(1, math.ceil(hp / 2)) end
elseif est > 0 and (string.find(w, "deagle", 1, true) or string.find(w, "desert", 1, true)) then if hp <= est then want = math.max(1, math.ceil(hp)) elseif hp <= (est * 2) then want = math.max(1, math.ceil(hp / 2)) end
end end end end if want ~= V.md_applied then if want == false then V.rage_reset() else V.api("rage.override_min_damage", rage.override_min_damage, want) end V.md_applied = want end end V.hitchance_tick = function()
if V.scope_blocked then return end local pm = V.hb_prio:get() if pm == "Menu" and V.res_defbody:get() and V.res_on:get() then local now = V.now_s() local tgt = V.cur_tgt if tgt and tgt.player then
local s = V.st(tgt.player) if s and (s.cls == "DEFENSIVE" or s.pdef) then V.def_body_until = now + 2 end end if now < V.def_body_until then pm = "Body First" end end
if pm == "Menu" and V.baim_pref then pm = "Body First" end if pm ~= V.prio_applied then if pm == "Menu" then if V.prio_applied then V.rage_reset() end else
V.api("rage.override_hitbox_priority", rage.override_hitbox_priority, V.PRIO[pm]) end V.prio_applied = pm end local spw = V.sp_force:get() if spw ~= V.sp_applied then
V.api("rage.override_force_safe_point", rage.override_force_safe_point, spw) V.sp_applied = spw end local hc = false if V.hc_on:get() then hc = V.hc_base:get() local cur = V.cur_tgt if cur and cur.player then
local s = V.st(cur.player) if s then hc = math.min(V.hc_max:get(), hc + (s.spread_miss * V.hc_step:get())) end end end if V.mhc_on:get() then local mv = V.move_hc() if mv ~= false then hc = mv end end if hc == false then
if V.hc_applied then V.rage_reset() end return end if hc ~= V.hc_applied then V.api("rage.override_hitchance", rage.override_hitchance, hc) V.hc_applied = hc end end client.set_event_callback("aim_fire", function(e)
local t = V.now_tick() V.DT.shot_t = V.now_s() V.shift_ready_at = t + 32 if V.aa_on:get() and V.def_on:get() and V.def_onshot:get() then V.flick_until = t + V.def_len:get() + 1
V.flick_sign = (math.random(0, 1) == 1) and 1 or -1 V.flick_pitch = (math.random(1, 4) == 1) and 0 or 89 end if V.aa_on:get() and V.def_spin:get() then V.spin_until = V.now_s() + (V.def_spin_ms:get() / 1000) end
local s = V.st(e.target) if s then s.shot_off = s.offset s.shot_anim = (s.anim_on == true) or (s.anim_off and s.offset == s.anim_off and V.res_mode:get() == "Lua Brute") or false s.shot_state = V.last_state
local ry = tonumber(e.resolved_yaw) if ry then s.res_yaw = ry end end end) client.set_event_callback("aim_hit", function(e) V.kill_flag = false V.stats.hits = V.stats.hits + 1 local function add_kill()
V.stats.kills = V.stats.kills + 1 V.stats.streak = V.stats.streak + 1 if V.stats.streak > V.stats.best_streak then V.stats.best_streak = V.stats.streak end V.kill_flag = true
local hs_kill = string.lower(tostring(e.hitbox)) == "head" pcall(V.on_kill_fx, hs_kill, tostring(V.key_of(e.target) or "")) pcall(V.bees_kill, e.target, hs_kill) pcall(V.killsay, hs_kill) end local sk = V.st(e.target)
local dmg = tonumber(e.damage) or 0 local hr = tonumber(e.health_remaining) if hr ~= nil and hr <= 0 then add_kill() if sk then sk.hp = 100 end elseif sk then local okh, h = pcall(entity.get_health, e.target)
local fresh = (okh and type(h) == "number") and h or nil local known = sk.hp or 100 local applied = sk.hp_t and ((V.now_s() - sk.hp_t) < 0.35) if fresh ~= nil and fresh <= 0 then add_kill(); sk.hp = 100
elseif fresh ~= nil and applied then sk.hp = fresh elseif fresh ~= nil and fresh < known then sk.hp = fresh else local cur = fresh or known if dmg >= cur then add_kill(); sk.hp = 100 else sk.hp = cur - dmg end end end
local hb = string.lower(tostring(e.hitbox)) local hp_left = sk and sk.hp or 100 local killed = V.kill_flag == true if not killed and dmg > 0 then V.body_learn(hb, dmg) end local hsub = hb
if sk and V.cls_label(sk) then hsub = hsub .. "  ·  vs " .. V.cls_label(sk) end if sk and sk.res_yaw then hsub = hsub .. "  ·  res " .. tostring(math.floor(V.norm(sk.res_yaw) + 0.5)) .. "°" end V.toast_add({
badge = killed and "KILL" or (hb == "head" and "HEAD" or "HIT"), col = killed and color(255, 90, 110) or V.accent_col:get() or color(0, 200, 255), title = tostring(V.key_of(e.target)), sub = hsub,
big = "-" .. tostring(math.floor(dmg + 0.5)), hpk = killed and 0 or (math.max(0, hp_left) / 100), }) local s = sk if not s then return end s.hits_n = s.hits_n + 1 pcall(V.anim_result, s, true)
if V.soft_hit(s, hb, dmg, killed, e.target) then return end s.soft = 0 s.hit_off = s.shot_off or s.offset V.ascore_add(s, s.hit_off, 2) s.misses = 0 s.spread_miss = 0 if V.log_shots:get() then
V.log(0, 255, 120, string.format("[NP] HIT %s | %s | dmg %s | hr %s | off %d | %s | %s%s", V.key_of(e.target), tostring(e.hitbox), tostring(e.damage), tostring(e.health_remaining),
math.floor(s.hit_off or 0), s.shot_state or V.last_state, tostring(V.cls_label(s) or "-"), "")) end end) client.set_event_callback("aim_miss", function(e) local reason = string.lower(tostring(e.reason))
local s = V.st(e.target) local tname = tostring(V.key_of(e.target)) if string.find(reason, "death", 1, true) or string.find(reason, "dead", 1, true) then V.stats.dead = V.stats.dead + 1
if s then s.dead_n = s.dead_n + 1 end V.toast_add({ badge = "LATE", col = V.C_DEAD_T, title = tname, sub = "target died before the bullet landed", big = "", hpk = -1 }) if V.log_shots:get() then
V.log(160, 160, 180, "[NP] MISS " .. tname .. " | " .. tostring(e.hitbox) .. " | death (not counted)") end return end V.stats.misses = V.stats.misses + 1 local kind = "other"
if reason == "resolver" then kind = "resolver" elseif reason == "spread" then kind = "spread" elseif string.find(reason, "occ", 1, true) then kind = "occlusion" end local bcol = color(255, 85, 95)
if kind == "spread" then bcol = color(255, 200, 60) elseif kind == "occlusion" then bcol = color(165, 120, 255) elseif kind == "other" then bcol = V.C_DEAD_T end local msub = string.lower(tostring(e.hitbox))
if s and V.cls_label(s) then msub = msub .. "  ·  vs " .. V.cls_label(s) end if s and s.res_yaw then msub = msub .. "  ·  res " .. tostring(math.floor(V.norm(s.res_yaw) + 0.5)) .. "°" end
V.toast_add({ badge = "MISS", col = bcol, title = tname, sub = msub, big = reason, hpk = -1 }) if not s then return end s.miss_n = s.miss_n + 1 if kind == "resolver" then V.stats.res_miss = V.stats.res_miss + 1
local now = V.now_s() if s.last_res_miss and (now - s.last_res_miss) > 10 then s.misses = 0 end s.last_res_miss = now s.misses = s.misses + 1 s.total_res = (s.total_res or 0) + 1 pcall(V.anim_result, s, false)
if V.bi_cycle:get() == "On Miss" and V.res_mode:get() ~= "Lua Brute" then V.bi_next_mode() end if V.hard_on:get() and not s.hard and s.total_res >= V.hard_after:get() then s.hard = true
V.log(255, 120, 0, "[NP] HARD TARGET: " .. tname .. " -> body aim from first shot") V.toast_push("HARD", color(255, 120, 0), tname .. "  ·  body aim") end s.stage = s.stage + 1 V.ascore_add(s, s.shot_off, -1)
if s.hit_off ~= nil and s.shot_off == s.hit_off then s.hit_off = nil end if V.res_on:get() and e.target ~= nil then local sp = V.sp_after:get() if V.want_baim(s) then if V.baim_mode:get() == "Force" then
V.api("plist.set", plist and plist.set, e.target, "Override prefer body aim", "Force") end end if sp > 0 and s.misses >= sp then V.api("plist.set", plist and plist.set, e.target, "Override safe point", "Force") end end
elseif kind == "spread" then V.stats.spread = V.stats.spread + 1 s.spread_miss = s.spread_miss + 1 elseif kind == "occlusion" then V.stats.occ = V.stats.occ + 1 else V.stats.other = V.stats.other + 1 end
if V.log_shots:get() then local r, g, b = 255, 80, 80 if kind == "spread" then r, g, b = 255, 200, 0 elseif kind == "occlusion" then r, g, b = 165, 120, 255 end
V.log(r, g, b, string.format("[NP] MISS %s | %s | %s | off %d | res-miss %d | %s | %s", tname, tostring(e.hitbox), reason,
math.floor(s.shot_off or 0), s.misses, s.shot_state or V.last_state, tostring(V.cls_label(s) or "-"))) end end) V.PF = { acc = {}, cnt = {}, frames = 0, ticks = 0, t = -10, wr = 0 } V.WR = 0 V.pf0 = function()
if not V.prof_on:get() then return false end return os.clock() end V.pf1 = function(k, t0) if not t0 then return end local a = V.PF.acc a[k] = (a[k] or 0) + (os.clock() - t0) end
V.PF_ORDER = { "paint", "labels", "players", "dtbar", "player", "toasts", "announcer", "bees", "grenades", "menu", "t_aa", "t_resolver", "t_rage" } V.pf_report = function(now) local P = V.PF if not V.prof_on:get() then
P.t = now P.frames = 0 P.ticks = 0 P.acc = {} V.WR = 0 return end P.frames = P.frames + 1 if (now - P.t) < 5 then return end local fr = math.max(1, P.frames) local tk = math.max(1, P.ticks) local parts = {}
for _, k in ipairs(V.PF_ORDER) do local v = P.acc[k] if v then local per = (string.sub(k, 1, 2) == "t_") and ((v / tk) * 1000) or ((v / fr) * 1000) parts[#parts + 1] = k .. " " .. string.format("%.2f", per) end end
V.log(150, 220, 255, "[NP] perf ms (frame / tick): " .. table.concat(parts, " | ") .. " | writes/frame " .. tostring(math.floor(V.WR / fr)) .. " | fps " .. tostring(math.floor(V.fps_s))) P.t = now P.frames = 0
P.ticks = 0 P.acc = {} V.WR = 0 end events.on_tick(function() V.tick_counter = V.tick_counter + 1 V.PF.ticks = V.PF.ticks + 1 local t = V.now_tick() local okt, tg0 = V.api("rage.get_target", rage.get_target)
V.cur_tgt = okt and tg0 or nil if V.cur_tgt and V.cur_tgt.player then local ry = tonumber(V.cur_tgt.resolved_yaw_deg) if ry then local s = V.st(V.cur_tgt.player) if s then s.res_yaw = ry end end end local q0 = V.pf0()
local ok1, e1 = pcall(V.aa_tick, t) if not ok1 then V.report("tick:aa", e1) end V.pf1("t_aa", q0) local ph = V.tick_counter % 4 if ph == 0 or ph == 2 then local q1 = V.pf0() local ok2, e2 = pcall(V.resolver_tick)
if not ok2 then V.report("tick:resolver", e2) end V.pf1("t_resolver", q1) end local q2 = V.pf0() if ph == 1 then local ok7, e7 = pcall(V.baim_tick) if not ok7 then V.report("tick:baim", e7) end
local ok3, e3 = pcall(V.hitchance_tick) if not ok3 then V.report("tick:hitchance", e3) end local ok8, e8 = pcall(V.md_tick) if not ok8 then V.report("tick:mindmg", e8) end end if ph == 1 or ph == 3 then
local ok5, e5 = pcall(V.scope_tick) if not ok5 then V.report("tick:scope", e5) end end if ph == 3 then local ok6, e6 = pcall(V.bi_cycle_tick) if not ok6 then V.report("tick:bi_cycle", e6) end end V.pf1("t_rage", q2) end)
V.C_BG = color(15, 17, 23) V.C_BG2 = color(27, 31, 40) V.C_BORDER = color(33, 37, 47) V.C_TEXT = color(236, 240, 255) V.C_DIM = color(118, 126, 145) V.C_SEP = color(44, 49, 61) V.C_GOOD = color(90, 220, 140)
V.C_BAD = color(255, 85, 95) V.C_WARN = color(255, 170, 60) V.C_WHITE = color(255, 255, 255) V.REG = {} V.REG_N = _v(0) V.reg = function(d) V.REG_N = V.REG_N + 1; V.REG[V.REG_N] = d; return d end V.Z_SHADOW = _v(10)
V.Z_BG = _v(11) V.Z_DECO = _v(12) V.Z_TEXT = _v(13) V.Z_TOAST_BG = _v(20) V.Z_TOAST_DECO = _v(21) V.Z_TOAST_TEXT = _v(22) V.Z_CROSS = _v(30) V.ZOF = {} V.setz = function(d, z) pcall(function() d.ZIndex = z end) end
V.FONTS = { ["Gotham"] = { "GothamMedium", "GothamBold" }, ["Builder Sans"] = { "BuilderSansMedium", "BuilderSansBold" }, ["Montserrat"] = { "MontserratMedium", "MontserratBold" }, ["Ubuntu"] = { "Ubuntu", "Ubuntu" },
["Classic UI"] = { 0, 0 }, ["Plex"] = { 2, 2 }, } V.TEXTS = {} V.TEXTS_N = _v(0) V.FONT_ENUM_OK = _v(true) V.TWC = {} V.TWC_N = _v(0) V.font_name = function(bold) local p = V.FONTS[V.font_sel:get()] or V.FONTS["Gotham"]
return bold and p[2] or p[1] end V.font_value = function(bold) local f = V.font_name(bold) if type(f) == "string" then if not V.FONT_ENUM_OK then return 0 end local ok, e = pcall(function() return Enum.Font[f] end)
if ok and e then return e end return 0 end return f end V.font_apply = function(d, bold) local fv = V.font_value(bold) local ok = pcall(function() d.Font = fv end) if not ok and type(fv) ~= "number" then
V.FONT_ENUM_OK = false pcall(function() d.Font = 0 end) end end V.font_refresh = function() V.TWC = {} V.TWC_N = 0 V.m_c2s = false for ti = 1, V.UI_TN do local sec = V.UI_TABS[ti] for j = 1, sec.n do
sec.items[j].lw0 = nil sec.items[j].vs_c = nil end end for i = 1, V.TEXTS_N do local e = V.TEXTS[i] V.font_apply(e.d, e.b) end end V.mk_text = function(size, outline, center, z, bold) local d = Drawing.new("Text")
d.Size = size V.font_apply(d, bold) V.TEXTS_N = V.TEXTS_N + 1 V.TEXTS[V.TEXTS_N] = { d = d, b = bold and true or false } d.Visible = false pcall(function() d.Center = center and true or false end)
pcall(function() d.Outline = true end) pcall(function() d.OutlineColor = color(0, 0, 0) end) V.ZOF[d] = z or V.Z_TEXT V.setz(d, V.ZOF[d]) return V.reg(d) end V.mk_rect = function(round, z) local d = Drawing.new("Square")
d.Rounding = round or 0 d.Visible = false pcall(function() d.Filled = true end) V.ZOF[d] = z or V.Z_BG V.setz(d, V.ZOF[d]) return V.reg(d) end V.pc = function(d)
if not V.PC[d] then V.PC[d] = { vis = d.Visible == true } end return V.PC[d] end V.hide = function(d) local c = V.PC[d] or V.pc(d) if c.vis == false then return end c.vis = false d.Visible = false V.WR = V.WR + 1 end
V.set_border = function(d, bc) d.BorderColor = bc end V.rect = function(d, x, y, w, h, col, border) local c = V.PC[d] or V.pc(d) local fx = math.floor(x) local fy = math.floor(y) if c.x ~= fx or c.y ~= fy then
d.Position = vector(fx, fy) V.WR = V.WR + 1 c.x = fx c.y = fy end local fw = math.max(0, math.floor(w)) local fh = math.max(0, math.floor(h)) if c.w ~= fw or c.h ~= fh then d.Size = vector(fw, fh) V.WR = V.WR + 1
c.w = fw c.h = fh end if c.col ~= col then d.Color = col V.WR = V.WR + 1 c.col = col end local bc = border or col if c.bc ~= bc then pcall(V.set_border, d, bc) c.bc = bc end if not c.vis then d.Visible = true
c.vis = true end end V.text = function(d, x, y, str, col) local c = V.PC[d] or V.pc(d) if c.str ~= str then d.Text = str V.WR = V.WR + 1 c.str = str end if c.col ~= col then d.Color = col V.WR = V.WR + 1 c.col = col end
local fx = math.floor(x) local fy = math.floor(y) if c.x ~= fx or c.y ~= fy then d.Position = vector(fx, fy) V.WR = V.WR + 1 c.x = fx c.y = fy end if not c.vis then d.Visible = true c.vis = true end end
V.potato = _v(false) V.set_grad = function(d, g) d.Gradient = g end V.gradA = function(d, c1, c2, ang, t1, t2) if V.potato then return end local c = V.PC[d] or V.pc(d) local a = ang or 0
if c.g1 == c1 and c.g2 == c2 and c.ga == a and c.gt1 == t1 and c.gt2 == t2 and c.gon then return end c.g1 = c1 c.g2 = c2 c.ga = a c.gt1 = t1 c.gt2 = t2 c.gon = true if t1 ~= nil then
pcall(V.set_grad, d, { c1, c2, a, t1, t2 }) else pcall(V.set_grad, d, { c1, c2, a }) end end V.grad = function(d, c1, c2) V.gradA(d, c1, c2, 0) end V.grad_off = function(d) local c = V.PC[d] or V.pc(d)
if c.gon == false then return end c.gon = false c.g1 = false pcall(V.set_grad, d, nil) end V.set_alpha = function(d, a) d.Transparency = a; V.WR = V.WR + 1 end V.alpha = function(d, a) local c = V.PC[d] or V.pc(d)
if c.a == a then return end c.a = a pcall(V.set_alpha, d, a) end V.kb_layout = _v(nil) V.TSVC = false pcall(function() V.TSVC = game:GetService("TextService") end) V.tw_enum = function(str, size)
local fname = V.font_name(false) if type(fname) ~= "string" or not V.FONT_ENUM_OK or not V.TSVC then return false end local key = fname .. "|" .. tostring(size) .. "|" .. str local c = V.TWC[key] if c then return c end
local ok, w = pcall(function() return V.TSVC:GetTextSize(str, size, Enum.Font[fname], Vector2.new(4000, 4000)).X end) if not (ok and type(w) == "number" and w > 0) then return false end if V.TWC_N > 600 then V.TWC = {}
V.TWC_N = 0 end V.TWC[key] = w V.TWC_N = V.TWC_N + 1 return w end V.tw_raw = function(str, size) if str == nil then return 0 end str = tostring(str) local we = V.tw_enum(str, size) if we then return we end
local fv = V.font_name(false) if type(fv) ~= "number" then fv = 2 end local ok, v = pcall(render.measure_text, str, size, fv) if ok and v then local okx, w = pcall(function() return v.X end)
if okx and type(w) == "number" and w > 0 then return w end local oky, w2 = pcall(function() return v.x end) if oky and type(w2) == "number" and w2 > 0 then return w2 end end return string.len(str) * size * 0.55 end
V.tb_get = function(d) return d.TextBounds.X end V.tw_obj = function(d, str, size) local ok, w = pcall(V.tb_get, d) if ok and type(w) == "number" and w > 0 then return w end return V.tw(str, size) end
V.tw = function(str, size) if V.tw_enum(tostring(str or ""), size) then return V.tw_raw(str, size) end return V.tw_raw(str, size) * 1.1 end V.font_sel:set_callback(function() V.font_refresh() end)
V.approach = function(cur, target, dt, speed) local k = dt * speed if k > 1 then k = 1 end return cur + (target - cur) * k end V.c_lerp = function(a, b, k) return a:Lerp(b, k) end V.MIXC = {} V.MIXC_N = 0
V.mix = function(a, b, k) if k <= 0.002 then return a end if k >= 0.998 then return b end local q = math.floor((k * 64) + 0.5) local ca = V.MIXC[a] if not ca then ca = {} V.MIXC[a] = ca end local cb = ca[b]
if not cb then cb = {} ca[b] = cb end local r = cb[q] if r then return r end local ok, rr = pcall(V.c_lerp, a, b, q / 64) if not ok or not rr then return (k > 0.5) and b or a end V.MIXC_N = V.MIXC_N + 1
if V.MIXC_N > 4000 then V.MIXC = {} V.MIXC_N = 0 end cb[q] = rr return rr end V.lighten = function(c, a) return V.mix(c, V.C_WHITE, a) end V.fps_s = _v(60) V.fps_n = _v(0) V.rs_frames = _v(0) V.rs_dt = _v(0)
V.fps_src_logged = _v(false) V.rs_conn = false pcall(function() V.rs_conn = game:GetService("RunService").RenderStepped:Connect(function(rdt) V.rs_frames = V.rs_frames + 1
if type(rdt) == "number" and rdt > 0 and rdt < 1 then V.rs_dt = V.rs_dt + rdt end end) end) V.fps_t0 = _v(0) V.ping_ms = _v(nil) V.dt_t = _v(0) V.dt_ok = _v(false) V.dt_charged = _v(nil) V.panels_t = _v(0) V.tw_t = _v(0)
V.fx_t = _v(0) V.panels_anim = _v(true) V.my_name = _v("") V.name_t = _v(0) V.clock_s = _v("") V.wm_shadow = V.mk_rect(6, V.Z_SHADOW) V.wm_bg = V.mk_rect(5, V.Z_BG) V.wm_line = V.mk_rect(1, V.Z_DECO)
V.mk_icon = function() return { V.mk_rect(2, V.Z_DECO + 1), V.mk_rect(2, V.Z_DECO + 1), V.mk_rect(2, V.Z_DECO + 1), V.mk_rect(2, V.Z_DECO + 1), V.mk_rect(2, V.Z_DECO + 1), V.mk_rect(2, V.Z_DECO + 1) } end
V.icon_hide = function(ic) ic.dn = false; V.hide(ic[1]); V.hide(ic[2]); V.hide(ic[3]); V.hide(ic[4]); V.hide(ic[5]); V.hide(ic[6]) end V.wm_logo = V.mk_rect(5, V.Z_DECO) V.wm_logo_in = V.mk_rect(4, V.Z_DECO + 1)
V.wm_logo_dot = V.mk_rect(2, V.Z_DECO + 2) V.wm_shine = V.mk_rect(1, V.Z_DECO + 1) V.st_shine = V.mk_rect(1, V.Z_DECO + 1) V.kb_shine = V.mk_rect(1, V.Z_DECO + 1) V.pl_shine = V.mk_rect(1, V.Z_DECO + 1)
V.st_ic = V.mk_icon() V.kb_ic = V.mk_icon() V.pl_ic = V.mk_icon() V.wm_logo_t = V.mk_text(12, false, true, V.Z_TEXT + 1, true) V.wm_t2 = V.mk_text(14, false, false, V.Z_TEXT, true)
V.wm_t1 = V.mk_text(14, false, false, V.Z_TEXT + 1, true) V.SEG_N = _v(5) V.wm_seg = {} V.wm_sep = {} for i = 1, V.SEG_N do V.wm_seg[i] = V.mk_text(13, false, false) V.wm_sep[i] = V.mk_rect(0, V.Z_DECO) end
V.wm_w = _v(200) V.st_shadow = V.mk_rect(6, V.Z_SHADOW) V.st_bg = V.mk_rect(5, V.Z_BG) V.st_line = V.mk_rect(1, V.Z_DECO) V.st_title = V.mk_text(11, false, false) V.st_pct = V.mk_text(11, false, false)
V.st_barbg = V.mk_rect(2, V.Z_DECO) V.st_bar = V.mk_rect(2, V.Z_TEXT) V.st_k1 = V.mk_text(12, false, false) V.st_v1 = V.mk_text(12, false, false) V.st_k2 = V.mk_text(12, false, false)
V.st_v2 = V.mk_text(12, false, false) V.st_misc = V.mk_text(11, false, false) V.st_k3 = V.mk_text(12, false, false) V.st_v3 = V.mk_text(12, false, false) V.st_segbg = V.mk_rect(2, V.Z_DECO) V.st_seg = {}
for i = 1, 5 do V.st_seg[i] = V.mk_rect(0, V.Z_TEXT) end V.SEG_COL = { color(255, 85, 95), color(255, 200, 60), color(165, 120, 255), color(120, 128, 150), color(70, 76, 98) } V.st_w = _v(220) V.bar_k = _v(0)
V.KB_N = _v(10) V.kb_shadow = V.mk_rect(6, V.Z_SHADOW) V.kb_bg = V.mk_rect(5, V.Z_BG) V.kb_line = V.mk_rect(1, V.Z_DECO) V.kb_title = V.mk_text(11, false, false) V.kb_name = {} V.kb_state = {} V.kb_pill = {}
V.kb_dot = {} V.KB_PLAIN = color(28, 33, 50) for i = 1, V.KB_N do V.kb_name[i] = V.mk_text(12, false, false) V.kb_state[i] = V.mk_text(12, false, false) V.kb_pill[i] = V.mk_rect(4, V.Z_DECO)
V.kb_dot[i] = V.mk_rect(3, V.Z_DECO + 1) end V.kb_w = _v(150) V.CI_N = _v(7) V.ci_txt = {} for i = 1, V.CI_N do V.ci_txt[i] = V.mk_text(12, true, true, V.Z_CROSS) end V.ci_arrow_l = V.mk_text(20, true, true, V.Z_CROSS)
V.ci_arrow_r = V.mk_text(20, true, true, V.Z_CROSS) V.TN = _v(6) V.TOAST_LIFE = _v(1.5) V.TL = { w = 248, h = 34, gap = 5 } V.TLC = {
bg = color(11, 13, 19), border = color(34, 38, 50), text = color(222, 228, 240), dim = color(122, 130, 150), track = color(30, 34, 44), } V.TQ = {} V.TQ_N = _v(0) V.kill_flag = _v(false) V.tl = {} for i = 1, V.TN do
V.tl[i] = { sh = V.mk_rect(10, V.Z_TOAST_BG - 1), bg = V.mk_rect(8, V.Z_TOAST_BG), bar = V.mk_rect(2, V.Z_TOAST_DECO), pill = V.mk_rect(4, V.Z_TOAST_DECO), ptx = V.mk_text(9, false, true, V.Z_TOAST_TEXT, true),
title = V.mk_text(12, false, false, V.Z_TOAST_TEXT, true), sub = V.mk_text(10, false, false, V.Z_TOAST_TEXT), big = V.mk_text(15, false, false, V.Z_TOAST_TEXT, true), hpbg = V.mk_rect(1, V.Z_TOAST_DECO),
hp = V.mk_rect(1, V.Z_TOAST_DECO + 1), life = V.mk_rect(1, V.Z_TOAST_DECO + 1), } end V.toast_add = function(e) if not V.show_toasts:get() then return end e.born = V.now_s() e.y = false
local n = math.min(V.TQ_N + 1, V.TN) for i = n, 2, -1 do V.TQ[i] = V.TQ[i - 1] end V.TQ[1] = e V.TQ_N = n end V.toast_push = function(badge, bcol, str)
V.toast_add({ badge = badge, col = bcol, title = str or "", sub = "", big = "", hpk = -1 }) end V.toasts_compact = function(now) local j = 0 for i = 1, V.TQ_N do local e = V.TQ[i] if (now - e.born) < V.TOAST_LIFE then
j = j + 1 V.TQ[j] = e end end V.TQ_N = j end V.hide_toast = function(i) for _, d in pairs(V.tl[i]) do V.hide(d) end end V.draw_toasts = function(now, scx, scy, tmax, dt) V.toasts_compact(now) if V.TQ_N == 0 then
if V.toasts_vis then for i = 1, V.TN do V.hide_toast(i) end V.toasts_vis = false end return end V.toasts_vis = true local W = V.TL.w local H = V.TL.h local life = V.TOAST_LIFE for i = 1, V.TN do local o = V.tl[i]
local e = V.TQ[i] if i <= V.TQ_N and i <= tmax and e then local age = now - e.born local target_y = (scy + 168) + ((i - 1) * (H + V.TL.gap)) if not e.y then e.y = target_y + 10 end
e.y = e.y + ((target_y - e.y) * math.min(1, dt * 16)) local inn = V.ease(age / 0.16) local out = V.ease((age - (life - 0.25)) / 0.25) local a = inn * (1 - out) local x = ((scx - (W / 2)) + ((1 - inn) * -18)) + (out * 18)
local y = e.y local col = e.col or V.C_WHITE V.rect(o.sh, x - 2, y - 1, W + 4, H + 4, V.BLACK) V.alpha(o.sh, 0.25 * a) V.rect(o.bg, x, y, W, H, V.TLC.bg, V.TLC.border) V.alpha(o.bg, 0.88 * a)
V.rect(o.bar, x + 4, y + 6, 2, H - 12, col) V.alpha(o.bar, a) local bw = V.tw(e.badge, 9) + 10 V.rect(o.pill, x + 12, y + 6, bw, 12, V.mix(V.TLC.bg, col, 0.3)) V.alpha(o.pill, a)
V.text(o.ptx, (x + 12) + (bw / 2), y + 6, e.badge, col) V.alpha(o.ptx, a) V.text(o.title, (x + 17) + bw, y + 5, e.title, V.TLC.text) V.alpha(o.title, a) V.text(o.sub, x + 12, y + 20, e.sub or "", V.TLC.dim)
V.alpha(o.sub, a) if e.big and e.big ~= "" then V.text(o.big, ((x + W) - 10) - V.tw(e.big, 15), y + 4, e.big, col) V.alpha(o.big, a) else V.hide(o.big) end if e.hpk and e.hpk >= 0 then local bwid = 52
local bx = ((x + W) - 10) - bwid V.rect(o.hpbg, bx, y + 25, bwid, 2, V.TLC.track) V.alpha(o.hpbg, a) local hk = math.max(0, math.min(1, e.hpk)) local hc = (hk > 0.6) and V.C_GOOD or ((hk > 0.25) and V.C_WARN or V.C_BAD)
if hk > 0.01 then V.rect(o.hp, bx, y + 25, math.max(2, bwid * hk), 2, hc) V.alpha(o.hp, a) else V.hide(o.hp) end else V.hide(o.hpbg); V.hide(o.hp) end local lk = 1 - math.min(1, age / life)
V.rect(o.life, x + 8, (y + H) - 2, (W - 16) * lk, 1, col) V.alpha(o.life, 0.5 * a) else V.hide_toast(i) end end end V.IN_START = _v(nil) V.INTRO_SLOW = _v(1.5) V.INTRO_LEN = 2.6 * V.INTRO_SLOW
V.LOGO = { "N", "E", "V", "E", "R", "P", "E", "E", "K" } V.BLACK = color(0, 0, 0) V.in_dim = { V.mk_rect(0, 35), V.mk_rect(0, 35) } V.in_vig = {} for i = 1, 8 do V.in_vig[i] = V.mk_rect(0, 36) end
for i = 1, _v(2) do V.alpha(V.in_dim[i], 0.5) end for i = 1, 8 do V.alpha(V.in_vig[i], 0.5) end V.in_glow = V.mk_rect(3, 40) V.in_line = V.mk_rect(1, 41) V.in_let = {}
for i = 1, 9 do V.in_let[i] = V.mk_text(30, false, false, 42, true) end V.in_glw1 = {} V.in_glw2 = {} for i = 1, 9 do V.in_glw1[i] = V.mk_text(34, false, true, 41, true)
V.in_glw2[i] = V.mk_text(40, false, true, 40, true) pcall(function() V.in_glw1[i].Outline = false end) pcall(function() V.in_glw2[i].Outline = false end) end
V.RING_D = { { 1, 0, 1 }, { -1, 0, 1 }, { 0, 1, 1 }, { 0, -1, 1 }, { 0.71, 0.71, 0.6 }, { -0.71, 0.71, 0.6 }, { 0.71, -0.71, 0.6 }, { -0.71, -0.71, 0.6 } } V.in_ring = {} V.mk_ring = function() local r = {}
for k = 1, _v(8) do local t = V.mk_text(30, false, false, 41, true) pcall(V.set_prop, t, "Outline", false) r[k] = t end return r end for i = 1, 9 do V.in_ring[i] = V.mk_ring() end
V.in_bloom = { V.mk_rect(0, 37), V.mk_rect(0, 37), V.mk_rect(0, 38), V.mk_rect(0, 38), V.mk_rect(0, 39), V.mk_rect(0, 39) } V.BLOOM_L = { { 50, 0.08, 1.0, 0 }, { 32, 0.1, 0.8, 0 }, { 14, 0.14, 0.6, 0.35 } }
V.ring_hide = function(i) local r = V.in_ring[i] for k = 1, _v(8) do V.hide(r[k]) end end V.bloom_hide = function()
V.hide(V.in_bloom[1]); V.hide(V.in_bloom[2]); V.hide(V.in_bloom[3]); V.hide(V.in_bloom[4]); V.hide(V.in_bloom[5]); V.hide(V.in_bloom[6]) end V.in_sub = V.mk_text(12, false, true, 42)
V.in_pct = V.mk_text(11, false, true, 42) V.hint_t = V.mk_text(14, false, true, 45) V.HN = { t0 = false, out_t = false, gone = false, key_t = -10 } V.hn_sh = V.mk_rect(9, 44) V.hn_bg = V.mk_rect(7, 45)
V.hn_line = V.mk_rect(1, 46) V.hn_bar = V.mk_rect(1, 46) V.hn_shn = V.mk_rect(1, 47) V.hn_ksh = V.mk_rect(5, 46) V.hn_key = V.mk_rect(5, 47) V.hn_t1 = V.mk_text(13, false, false, 48)
V.hn_kt = V.mk_text(12, false, true, 48, true) V.hn_t2 = V.mk_text(13, false, false, 48) V.hn_dot = V.mk_rect(3, 48) V.hint_hide = function()
V.hide(V.hn_sh); V.hide(V.hn_bg); V.hide(V.hn_line); V.hide(V.hn_bar); V.hide(V.hn_shn) V.hide(V.hn_ksh); V.hide(V.hn_key); V.hide(V.hn_t1); V.hide(V.hn_kt); V.hide(V.hn_t2); V.hide(V.hn_dot) end
V.hn_part = function(str, frac) if frac <= 0 then return "" end if frac >= 1 then return str end return string.sub(str, 1, math.floor(string.len(str) * frac)) end V.draw_hint = function(now, acc, acc2) local H = V.HN
if H.gone then return end if V.ui_hint_done and not H.out_t then H.out_t = now end if not H.t0 then H.t0 = now end local age = now - H.t0 local outk = 0 if H.out_t then outk = V.ease((now - H.out_t) / 0.35)
if outk >= 1 then H.gone = true V.hint_hide() return end end local key = tostring(V.k_menu:get() or "RightShift") local s1 = "press" local s2 = "to open menu" local w1 = V.tw(s1, 13) local wk = V.tw(key, 12) + 18
local w2 = V.tw(s2, 13) local W = math.floor(((((22 + w1) + 10) + wk) + 10) + (w2 + 16)) local Hh = 32 local x = math.floor((V.sw / 2) - (W / 2)) local y = math.floor((V.sh / 2) - 78) local k = V.ease(age / 0.6)
local cw = math.max(8, (W * k) * (1 - outk)) local fade = 1 - outk V.rect(V.hn_sh, x - 3, y - 2, cw + 6, Hh + 6, V.BLACK) V.alpha(V.hn_sh, 0.28 * fade) V.rect(V.hn_bg, x, y, cw, Hh, V.C_BG, V.C_BORDER)
V.alpha(V.hn_bg, 0.95 * fade) if cw > 14 then V.rect(V.hn_line, x + 6, y, cw - 12, 2, acc) V.grad(V.hn_line, acc, acc2) V.alpha(V.hn_line, fade) else V.hide(V.hn_line) end local pulse = 0.5 + (0.5 * math.sin(now * 3.2))
V.rect(V.hn_bar, x + 8, y + 9, 3, Hh - 18, acc) V.alpha(V.hn_bar, (0.55 + (0.45 * pulse)) * fade) local edge = x + cw local tx1 = x + 20 local f1 = (edge - tx1) / math.max(1, w1) if f1 > 0 then
V.text(V.hn_t1, tx1, y + 9, V.hn_part(s1, f1), V.C_DIM) V.alpha(V.hn_t1, fade) else V.hide(V.hn_t1) end local kx = (tx1 + w1) + 10 local kreach = (edge - kx) / wk if kreach > 0.5 then
if H.key_t < H.t0 then H.key_t = now end local pop = V.ease_back((now - H.key_t) / 0.3) local cyc = (age % 2.2) local pressed = (age > 1) and (cyc < 0.14) local kh = 20 local kw = wk * (0.7 + (0.3 * pop))
local kxx = kx + ((wk - kw) / 2) local ky = (y + 6) + (pressed and 2 or 0) V.rect(V.hn_ksh, kxx, (y + 6) + 3, kw, kh, V.mix(V.C_BG, acc, 0.35)) V.alpha(V.hn_ksh, fade)
V.rect(V.hn_key, kxx, ky, kw, kh - (pressed and 1 or 0), pressed and V.mix(V.C_BG2, acc, 0.35) or V.C_BG2, pressed and acc2 or acc) V.alpha(V.hn_key, fade) if pop > 0.6 then
V.text(V.hn_kt, kxx + (kw / 2), ky + 3, key, pressed and V.C_WHITE or acc2) V.alpha(V.hn_kt, fade) else V.hide(V.hn_kt) end else V.hide(V.hn_ksh); V.hide(V.hn_key); V.hide(V.hn_kt) end local tx2 = (kx + wk) + 10
local f2 = (edge - tx2) / math.max(1, w2) if f2 > 0 then V.text(V.hn_t2, tx2, y + 9, V.hn_part(s2, f2), V.C_TEXT) V.alpha(V.hn_t2, fade) else V.hide(V.hn_t2) end if k < 1 and outk <= 0 then
V.rect(V.hn_dot, edge - 3, y + 8, 2, Hh - 16, acc2) V.alpha(V.hn_dot, 0.9) else V.hide(V.hn_dot) end local sp = ((age - 1.2) % 3.2) if age > 1.2 and sp < 0.7 and outk <= 0 then
local sx = (x + 6) + (((W - 12) - 34) * V.ease(sp / 0.7)) V.rect(V.hn_shn, sx, y, 34, 2, V.C_WHITE) V.alpha(V.hn_shn, 0.75 * math.sin((sp / 0.7) * math.pi)) else V.hide(V.hn_shn) end end V.ease = function(t)
if t <= 0 then return 0 end if t >= 1 then return 1 end local u = 1 - t return 1 - ((u * u) * u) end V.ease_back = function(t) if t <= 0 then return 0 end if t >= 1 then return 1 end local u = t - 1
return (1 + (2.70158 * ((u * u) * u))) + (1.70158 * (u * u)) end V.intro_hide = function() V.hide(V.in_glow); V.hide(V.in_line); V.hide(V.in_sub); V.hide(V.in_pct) for i = 1, _v(2) do V.hide(V.in_dim[i]) end
for i = 1, 8 do V.hide(V.in_vig[i]) end for i = 1, 9 do V.hide(V.in_let[i]); V.hide(V.in_glw1[i]); V.hide(V.in_glw2[i]); V.ring_hide(i) end V.bloom_hide() end V.intro_elapsed = function(now)
if not V.intro_on:get() or V.perf_mode:get() == "Potato" then return 99 end if not V.IN_START then V.IN_START = now end return now - V.IN_START end V.appear = function(el, start)
if el >= (start + 0.45) then return true, 0 end if el < start then return false, 0 end local e = V.ease_back((el - start) / 0.45) return true, -60 * (1 - e) end V.glow_letter = function(i, x, y, size, fade, gk, acc)
local r = V.in_ring[i] local gc = V.lighten(acc, 0.25) local rad = 1 + (2.5 * gk) for k = 1, _v(8) do local t = r[k] local dd = V.RING_D[k] if t.Size ~= size then t.Size = size end
V.text(t, x + (dd[1] * rad), y + (dd[2] * rad), V.LOGO[i], gc) V.alpha(t, 0.24 * dd[3] * fade) end end V.draw_bloom = function(cx, cy, w, fade, acc) if fade <= 0.01 then V.bloom_hide() return end local B = V.in_bloom
V.bloom_layer(B[1], B[2], cx, cy, w, V.BLOOM_L[1], fade, acc) V.bloom_layer(B[3], B[4], cx, cy, w, V.BLOOM_L[2], fade, acc) V.bloom_layer(B[5], B[6], cx, cy, w, V.BLOOM_L[3], fade, acc) end
V.bloom_layer = function(l, r, cx, cy, w, L, fade, acc) local hw = (w * L[3]) / 2 local h = L[1] local c = (L[4] > 0) and V.lighten(acc, L[4]) or acc local xl = math.floor(cx - hw) local wl = math.floor(hw)
local yy = math.floor(cy - (h / 2)) V.rect(l, xl, yy, wl, h, c) V.rect(r, xl + wl, yy, wl, h, c) V.gradA(l, c, c, 0, 1, 0) V.gradA(r, c, c, 0, 0, 1) V.alpha(l, L[2] * 2 * fade) V.alpha(r, L[2] * 2 * fade) end
V.intro_done = _v(false) V.draw_intro = function(el, acc, acc2) if el >= V.INTRO_LEN then if not V.intro_done then V.intro_done = true; V.intro_hide() end return end el = el / V.INTRO_SLOW
local cx0, cy0 = V.sw / 2, (V.sh / 2) - 50 if V.intro_dim:get() then local d1 = el > 0.05 and el < 2.45 local d2 = el > 0.2 and el < 2.3 local v1 = el > 0.3 and el < 2.2 local v2 = el > 0.42 and el < 2.1
if d1 then V.rect(V.in_dim[1], 0, 0, V.sw, V.sh, V.BLACK, V.BLACK) else V.hide(V.in_dim[1]) end if d2 then V.rect(V.in_dim[2], 0, 0, V.sw, V.sh, V.BLACK, V.BLACK) else V.hide(V.in_dim[2]) end
local bands = { { v1, 0.22, 0.16 }, { v2, 0.11, 0.08 } } for b = 1, _v(2) do local on, fh, fw = bands[b][1], bands[b][2], bands[b][3] local vh, vw = V.sh * fh, V.sw * fw local base = (b - 1) * 4 if on then
V.rect(V.in_vig[base + 1], 0, 0, V.sw, vh, V.BLACK, V.BLACK) V.rect(V.in_vig[base + 2], 0, V.sh - vh, V.sw, vh, V.BLACK, V.BLACK) V.rect(V.in_vig[base + 3], 0, 0, vw, V.sh, V.BLACK, V.BLACK)
V.rect(V.in_vig[base + 4], V.sw - vw, 0, vw, V.sh, V.BLACK, V.BLACK) else for j = 1, _v(4) do V.hide(V.in_vig[base + j]) end end end else for i = 1, _v(2) do V.hide(V.in_dim[i]) end
for i = 1, 8 do V.hide(V.in_vig[i]) end end local lw = 320 * V.ease((el - 0.25) / 0.5) lw = lw * (1 - V.ease((el - 2.0) / 0.35)) local ly = cy0 + 36 if lw > 1 then local dark = acc
pcall(function() dark = acc:Lerp(V.BLACK, 0.55) end) V.rect(V.in_glow, (cx0 - (lw / 2)) - 3, ly - 2, lw + 6, 6, dark) V.rect(V.in_line, cx0 - (lw / 2), ly, lw, 2, acc); V.grad(V.in_line, acc2, acc) else
V.hide(V.in_glow); V.hide(V.in_line) end local ws, total = {}, 0 for i = 1, 9 do ws[i] = V.tw_raw(V.LOGO[i], 30); total = total + ws[i] end total = total + (8 * 7) local gk = V.ease((el - 1.45) / 0.3)
local zk = 1 + (0.10 * V.ease((el - 1.45) / 0.6)) local lx = cx0 - ((total * zk) / 2) local glow_on = V.intro_glow_ok ~= false for i = 1, 9 do local ti = 0.55 + ((i - 1) * 0.07) local te = 1.95 + ((i - 1) * 0.035)
local d = V.in_let[i] local g1 = V.in_glw1[i] local g2 = V.in_glw2[i] local wi = ws[i] * zk V.hide(g1); V.hide(g2) if el < ti then V.hide(d); V.ring_hide(i) else local a = V.ease_back((el - ti) / 0.35)
local oy = (1 - a) * -22 local ex = V.ease((el - te) / 0.3) if ex >= 1 then V.hide(d); V.ring_hide(i) else local z = zk + (0.08 * ex) local size = math.floor((30 * z) + 0.5)
local ly0 = ((cy0 - 6) + oy) - ((size - 30) / 2) local base = (i <= 5) and V.C_WHITE or acc local col = base if gk > 0 then col = V.mix(base, V.lighten(acc, 0.6), gk * 0.35) end d.Size = size
local lx0 = lx + ((wi - (ws[i] * z)) / 2) V.text(d, lx0, ly0, V.LOGO[i], col) V.alpha(d, 1 - ex) if gk > 0.02 and glow_on then V.glow_letter(i, lx0, ly0, size, gk * (1 - ex), gk, acc) else V.ring_hide(i) end end end
lx = (lx + wi) + (7 * zk) end V.bloom_hide() if el > 1.15 and el < 2.05 then local full = "rage lua  ·  build 1.2" local n = math.floor(string.len(full) * V.ease((el - 1.15) / 0.4))
V.text(V.in_sub, cx0, ly + 12, string.sub(full, 1, n), V.C_DIM) local p = math.floor(100 * V.ease((el - 1.2) / 0.75)) if p >= 100 then V.text(V.in_pct, cx0, ly + 32, "ready", V.C_GOOD) else
V.text(V.in_pct, cx0, ly + 32, "loading  " .. tostring(p) .. "%", acc2) end else V.hide(V.in_sub); V.hide(V.in_pct) end end V.dragging = _v(false) V.drag_dx = _v(0) V.drag_dy = _v(0)
V.handle_drag = function(x0, y0, w, h) if not V.wm_drag:get() then V.dragging = false return end if not V.ui_open then V.dragging = false return end
local okp, mx, my = V.api("input.get_mouse_position", input.get_mouse_position) local okd, down = V.api("input.is_mouse_down", input.is_mouse_down, 1) if not (okp and okd and mx and my) then return end if down then
if not V.dragging then if mx >= x0 and mx <= x0 + w and my >= y0 and my <= y0 + h then V.dragging = true V.drag_dx = mx - x0 V.drag_dy = my - y0 end else V.wm_x:set(math.floor(mx - V.drag_dx), true)
V.wm_y:set(math.floor(my - V.drag_dy), true) end else V.dragging = false end end V.hide_range = function(list, n) for i = 1, n do V.hide(list[i]) end end V.PANEL_G1 = color(21, 25, 38) V.PANEL_G2 = color(10, 12, 19)
V.panel = function(shadow, bg, line, x, y, w, h, acc, acc2, shine, ic, icname) V.hide(shadow) V.grad_off(bg) V.rect(bg, x, y, w, h, V.C_BG, V.C_BORDER) V.alpha(bg, 0.95) V.hide(line) if shine then V.hide(shine) end
if ic and icname then V.draw_icon(ic, icname, x + 9, y + 7, acc) end end V.an_name = function(n) if n == 2 then return "DOUBLE KILL" end if n == 3 then return "TRIPLE KILL" end if n == 4 then return "QUAD KILL" end
if n == 5 then return "PENTA KILL" end if n >= 6 then return "GODLIKE  x" .. tostring(n) end return nil end V.AN_G1 = color(16, 19, 28) V.AN_G2 = color(8, 9, 14) V.LOGO_G1 = color(22, 26, 36)
V.LOGO_G2 = color(10, 12, 17) V.AN = { text = false, t = -10, hs = false, n = 0, victim = "", col = false, chip = false, sub = false, kind = "streak", label = "", rb = false } V.an_glow2 = V.mk_rect(24, 28)
V.an_glow1 = V.mk_rect(19, 29) V.an_plate_b = V.mk_rect(14, 30) V.an_plate = V.mk_rect(14, 31) V.an_top = V.mk_rect(1, 34) V.an_shine = V.mk_rect(3, 35) V.an_lab = V.mk_text(11, false, true, 36, true)
V.an_txt = V.mk_text(34, false, true, 36, true) V.an_ll = V.mk_rect(1, 34) V.an_lr = V.mk_rect(1, 34) V.an_chip = V.mk_rect(9, 35) V.an_chip_t = V.mk_text(13, false, true, 37, true)
V.an_sub = V.mk_text(12, false, true, 36) V.an_drain = V.mk_rect(1, 34) V.an_rt_ring = V.mk_rect(11, 35) V.an_rt_dot = V.mk_rect(2, 36) V.an_rt_a = V.mk_rect(0, 36) V.an_rt_b = V.mk_rect(0, 36)
V.an_rt_c = V.mk_rect(0, 36) V.an_rt_d = V.mk_rect(0, 36) V.an_pip = {} for i = 1, 6 do V.an_pip[i] = V.mk_rect(3, 35) end pcall(function() V.an_lab.Outline = false end) pcall(function() V.an_chip_t.Outline = false end)
V.AN_RET = { V.an_rt_ring, V.an_rt_dot, V.an_rt_a, V.an_rt_b, V.an_rt_c, V.an_rt_d }
V.AN_OBJS = { V.an_glow2, V.an_glow1, V.an_plate, V.an_plate_b, V.an_top, V.an_shine, V.an_lab, V.an_txt, V.an_ll, V.an_lr, V.an_chip, V.an_chip_t, V.an_sub, V.an_drain,
V.an_rt_ring, V.an_rt_dot, V.an_rt_a, V.an_rt_b, V.an_rt_c, V.an_rt_d } V.an_hide = function() for _, d in ipairs(V.AN_OBJS) do V.hide(d) end for i = 1, 6 do V.hide(V.an_pip[i]) end end V.AN_HS_COL = color(255, 78, 92)
V.an_tier_col = function(n, acc) if n >= 6 then return color(255, 205, 80) end if n == 5 then return color(255, 72, 96) end if n == 4 then return color(255, 150, 60) end if n == 3 then return color(175, 115, 255) end
return acc end V.an_label = function(n) if n >= 6 then return "UNSTOPPABLE" end if n == 5 then return "RAMPAGE" end if n >= 3 then return "KILLSTREAK" end return "MULTI KILL" end V.an_ret = function(cx, cy, tc, plate, a)
V.rect(V.an_rt_ring, cx - 11, cy - 11, 22, 22, plate, tc) V.rect(V.an_rt_dot, cx - 2, cy - 2, 4, 4, tc) V.rect(V.an_rt_a, cx - 1, cy - 16, 2, 6, tc) V.rect(V.an_rt_b, cx - 1, cy + 10, 2, 6, tc)
V.rect(V.an_rt_c, cx - 16, cy - 1, 6, 2, tc) V.rect(V.an_rt_d, cx + 10, cy - 1, 6, 2, tc) for _, d in ipairs(V.AN_RET) do V.alpha(d, a) end end V.draw_announcer = function(now, acc, acc2) local AN = V.AN
local age = now - AN.t local life = (AN.kind == "egg") and 3.2 or 2.6 if not AN.text or age > life then if V.an_txt.Visible then V.an_hide() end return end local kind = AN.kind
local tc = AN.col or V.an_tier_col(AN.n, acc) if kind == "hs" then tc = V.AN_HS_COL end if AN.rb then tc = V.hsv((now * 0.22) % 1, 0.5, 1) end local tl = V.lighten(tc, 0.45) local inn = V.ease_back(age / 0.32)
local out = V.ease((age - (life - 0.55)) / 0.5) if out >= 1 then V.an_hide() return end local fade = (1 - out) * math.min(1, age / 0.12) local cx0 = V.sw / 2 local pips = (kind == "streak") local ret = (kind == "hs")
local scale = 1 + ((1 - math.min(1, inn)) * 0.3) local size = math.floor((32 * scale) + 0.5) local tw0 = V.tw(AN.text, 32) local chip = AN.chip or ((kind == "hs") and "HS" or ("x" .. tostring(math.max(1, AN.n))))
local cw = V.tw(chip, 13) + 16 local lead = ret and 34 or 0 local cwid = (lead + tw0) + (cw + 14) local pw = math.max(260, cwid + 72) local ph = pips and 104 or 90
local py = ((V.sh * 0.2) - ((1 - inn) * 18)) - (out * 26) local px = cx0 - (pw / 2) local plate = color(10, 12, 18) local gp = 0.5 + (0.5 * math.sin(age * 4)) V.rect(V.an_glow2, px - 12, py - 12, pw + 24, ph + 24, tc)
V.alpha(V.an_glow2, (0.035 + (0.025 * gp)) * fade) V.rect(V.an_glow1, px - 6, py - 6, pw + 12, ph + 12, tc) V.alpha(V.an_glow1, (0.07 + (0.04 * gp)) * fade) V.rect(V.an_plate_b, px - 1, py - 1, pw + 2, ph + 2, tc)
V.alpha(V.an_plate_b, 0.35 * fade) V.rect(V.an_plate, px, py, pw, ph, plate) V.alpha(V.an_plate, 0.92 * fade) V.gradA(V.an_plate, V.mix(V.AN_G1, tc, 0.10), V.AN_G2, 90) local tk = V.ease((age - 0.05) / 0.45)
local tw2 = (pw - 40) * tk V.rect(V.an_top, cx0 - (tw2 / 2), py, tw2, 2, tc) V.gradA(V.an_top, tl, tc, 0) V.alpha(V.an_top, fade) local lab = AN.label
if lab == "" then lab = (kind == "hs") and "CLEAN KILL" or V.an_label(AN.n) end local lk = V.ease((age - 0.1) / 0.3) V.text(V.an_lab, cx0, (py + 10) - ((1 - lk) * 4), lab, V.mix(tc, V.C_WHITE, 0.25))
V.alpha(V.an_lab, 0.85 * lk * fade) local tx = (cx0 - (cwid / 2)) + lead + (tw0 / 2) local ty = (py + 25) - ((size - 32) / 2) V.an_txt.Size = size local flash = 1 - V.ease(age / 0.45)
V.text(V.an_txt, tx, ty, AN.text, V.mix(tc, V.C_WHITE, 0.4 + (0.6 * flash))) V.alpha(V.an_txt, fade) if ret then local rk = V.ease_back((age - 0.08) / 0.35) local rcx = (cx0 - (cwid / 2)) + 12 local rcy = py + 42
if rk > 0.01 then V.an_ret(rcx, rcy - ((1 - math.min(1, rk)) * 8), tc, plate, fade * math.min(1, rk)) else for _, d in ipairs(V.AN_RET) do V.hide(d) end end else for _, d in ipairs(V.AN_RET) do V.hide(d) end end
local sh = (age - 0.22) / 0.55 if sh > 0 and sh < 1 then local sx = (px + 10) + ((pw - 20) * V.ease(sh)) V.rect(V.an_shine, sx - 4, py + 4, 8, ph - 8, V.C_WHITE) V.alpha(V.an_shine, 0.09 * (1 - sh) * fade) else
V.hide(V.an_shine) end local chx = ((cx0 - (cwid / 2)) + lead + tw0) + 14 local cin = V.ease_back((age - 0.18) / 0.3) if cin > 0.01 then local cy = (py + 31) + ((1 - cin) * 10) V.rect(V.an_chip, chx, cy, cw, 22, tc)
V.gradA(V.an_chip, tl, tc, 90) V.alpha(V.an_chip, fade) V.text(V.an_chip_t, chx + (cw / 2), cy + 3, chip, color(12, 14, 20)) V.alpha(V.an_chip_t, fade) else V.hide(V.an_chip); V.hide(V.an_chip_t) end
local lw = ((pw / 2) - 30) * V.ease((age - 0.12) / 0.4) local ly = py + 64 if lw > 2 then V.rect(V.an_ll, cx0 - lw, ly, lw, 1, tc) V.rect(V.an_lr, cx0, ly, lw, 1, tc) V.grad(V.an_ll, plate, tc) V.grad(V.an_lr, tc, plate)
V.alpha(V.an_ll, 0.8 * fade) V.alpha(V.an_lr, 0.8 * fade) else V.hide(V.an_ll); V.hide(V.an_lr) end local sub = AN.sub if not sub then if kind == "hs" then
sub = (AN.victim ~= "") and ("one tap on " .. AN.victim) or "one tap" else sub = "streak " .. tostring(V.stats.streak) if AN.victim ~= "" then sub = AN.victim .. "  ·  " .. sub end
if AN.hs then sub = "headshot  ·  " .. sub end end end local sk = V.ease((age - 0.3) / 0.35) V.text(V.an_sub, cx0, (py + 70) + ((1 - sk) * 6), sub, V.mix(V.C_DIM, V.C_TEXT, 0.45)) V.alpha(V.an_sub, sk * fade)
local np = math.min(6, math.max(1, AN.n)) local pw0 = (np * 16) - 6 local px0 = cx0 - (pw0 / 2) for i = 1, 6 do local d = V.an_pip[i] if pips and i <= np then
local on = V.ease_back((age - (0.35 + ((i - 1) * 0.07))) / 0.25) if on > 0.01 then local ww = 6 + (4 * math.min(1, on)) V.rect(d, (px0 + ((i - 1) * 16)) + ((10 - ww) / 2), py + 89, ww, 4, (i == np) and tl or tc)
V.alpha(d, fade) else V.hide(d) end else V.hide(d) end end local dk = 1 - math.max(0, math.min(1, age / (life - 0.55))) local dw = (pw - 28) * dk if dw > 1 then
V.rect(V.an_drain, cx0 - (dw / 2), (py + ph) - 3, dw, 2, tc) V.alpha(V.an_drain, 0.45 * fade) else V.hide(V.an_drain) end end V.mk_count = _v(0) V.mk_last = _v(-100) V.on_kill_fx = function(headshot, victim)
local now = V.now_s() if (now - V.mk_last) <= V.ann_window:get() then V.mk_count = V.mk_count + 1 else V.mk_count = 1 end V.mk_last = now if not V.ann_on:get() then return end local t = V.an_name(V.mk_count) or false
if not t and headshot and V.ann_hs:get() then t = "HEADSHOT" end if not t then return end if type(t) ~= "string" then return end V.AN.text = t V.AN.t = now V.AN.hs = headshot V.AN.n = V.mk_count
V.AN.victim = tostring(victim or "") V.AN.chip = false V.AN.sub = false V.AN.col = false V.AN.rb = false V.AN.label = "" V.AN.kind = (V.mk_count >= 2) and "streak" or "hs" local snd = V.ann_sound:get()
if snd ~= "Off" then V.api("sounds.play", sounds and sounds.play, snd, 1) end end V.EL_N = _v(10) V.el_bg = {} V.el_tx = {} V.el_hb = {} V.el_ht = {} for i = 1, V.EL_N do V.el_bg[i] = V.mk_rect(4, 26)
V.el_tx[i] = V.mk_text(11, false, true, 28) V.el_hb[i] = V.mk_rect(3, 26) V.el_ht[i] = V.mk_text(10, false, true, 28) end V.el_data_t = _v(0) V.el_txt = {} V.el_hard = {} V.el_ent = {} V.hide_label = function(i)
V.hide(V.el_bg[i]); V.hide(V.el_tx[i]); V.hide(V.el_hb[i]); V.hide(V.el_ht[i]) end V.el_w = {} V.toasts_vis = true V.aa_released = false V.el_pos = {} V.el_pos_t = {} V.el_pos_e = {} V.draw_labels = function(now, acc)
if not V.lab_on:get() or not V.res_on:get() then for i = 1, V.EL_N do V.hide_label(i) end return end if (now - V.el_data_t) > 0.1 then V.el_data_t = now for i = 1, V.EL_N do local e = V.E_LIST[i]
V.el_ent[i] = (i <= V.E_N) and e or nil if V.el_ent[i] then local s = V.st(e)
local mode = (V.res_mode:get() == "Lua Brute") and (tostring(math.floor(s and s.offset or 0)) .. "°") or ((s and s.anim_on) and ("anim " .. tostring(s.offset) .. "°") or "built-in")
local cl = (s and s.cls and V.res_cls:get()) and (V.cls_label(s) .. "  ·  ") or "" local nt = cl .. mode .. "  ·  m" .. tostring(s and s.misses or 0) if s and s.jitter and not s.cls then nt = nt .. "  ·  jit" end
if nt ~= V.el_txt[i] then V.el_txt[i] = nt V.el_w[i] = V.tw(nt, 11) + 14 end V.el_hard[i] = s and s.hard end end end for i = 1, V.EL_N do local e = V.el_ent[i] local shown = false if e then local pos = V.el_pos[i]
if (now - (V.el_pos_t[i] or -10)) > 0.033 or V.el_pos_e[i] ~= e then local okp, pp = pcall(entity.get_player_hitbox_position, e, "head") pos = (okp and pp) or false V.el_pos[i] = pos V.el_pos_t[i] = now V.el_pos_e[i] = e
end if pos then local okw, x, y, on = pcall(render.world_to_screen, pos) if okw and on and type(x) == "number" then shown = true local t = V.el_txt[i] or "" local w = V.el_w[i] or (V.tw(t, 11) + 14) local py = y - 34
V.rect(V.el_bg[i], x - (w / 2), py, w, 16, V.C_BG, V.el_hard[i] and V.C_BAD or acc) V.text(V.el_tx[i], x, py + 2, t, V.C_TEXT) if V.el_hard[i] then V.rect(V.el_hb[i], x - 18, py - 15, 36, 13, V.C_BAD)
V.text(V.el_ht[i], x, py - 15, "HARD", V.C_WHITE) else V.hide(V.el_hb[i]); V.hide(V.el_ht[i]) end end end end if not shown then V.hide_label(i) end end end V.PL_N = _v(10) V.PL_COLS = { 10, 136, 206, 258, 312, 362 }
V.PL_HEAD = { "NAME", "AA", "RES", "H / M", "DIST", "HP" } V.PL_W = _v(400) V.pl_shadow = V.mk_rect(6, V.Z_SHADOW) V.pl_bg = V.mk_rect(5, V.Z_BG) V.pl_line = V.mk_rect(1, V.Z_DECO)
V.pl_title = V.mk_text(11, false, false) V.pl_cnt = V.mk_text(11, false, false) V.pl_hsep = V.mk_rect(0, V.Z_DECO) V.pl_head = {} V.pl_hl_r = {} V.pl_cell = {}
for c = 1, 6 do V.pl_head[c] = V.mk_text(10, false, false) end for r = 1, V.PL_N do V.pl_hl_r[r] = V.mk_rect(3, V.Z_DECO) V.pl_cell[r] = {} for c = 1, 6 do V.pl_cell[r][c] = V.mk_text(12, false, false) end end
V.CLS_COL = { STATIC = color(90, 220, 140), JITTER = color(255, 170, 60), SPIN = color(165, 120, 255), DEFENSIVE = color(255, 85, 95), RANDOM = color(0, 220, 255), } V.pl_t = _v(0) V.pl_drag = _v(false) V.pl_dx = _v(0)
V.pl_dy = _v(0) V.pl_prev = _v(false) V.pl_rows = {} V.pl_rn = _v(0) V.pl_hide = function() V.hide(V.pl_shadow); V.hide(V.pl_bg); V.hide(V.pl_line); V.hide(V.pl_title); V.hide(V.pl_cnt); V.hide(V.pl_hsep)
V.hide(V.pl_shine); V.icon_hide(V.pl_ic) V.hide_range(V.pl_head, 6) for r = 1, V.PL_N do V.hide(V.pl_hl_r[r]) for c = 1, 6 do V.hide(V.pl_cell[r][c]) end end end V.pl_collect = function()
local tgt_name = (V.cur_tgt and V.cur_tgt.player) and V.key_of(V.cur_tgt.player) or nil local n, alive_n = 0, 0 local now = V.now_s() for i = 1, V.E_N do local s = V.st(V.E_LIST[i])
if s and s.seen_t and (now - s.seen_t) < 3 then if s.alive then alive_n = alive_n + 1 end if (s.alive or V.pl_dead:get()) and n < V.PL_N then n = n + 1 if not V.pl_rows[n] then V.pl_rows[n] = {} end
local row = V.pl_rows[n] local nm = tostring(s.name or "?") if string.len(nm) > 15 then nm = string.sub(nm, 1, 14) .. ".." end row.name = nm row.s = s row.tgt = (tgt_name ~= nil and s.name == tgt_name)
if not V.res_cls:get() then row.cls = "-" elseif s.cls then row.cls = V.cls_label(s) else row.cls = "..." end if V.res_mode:get() == "Lua Brute" then row.res = tostring(math.floor(s.offset or 0)) .. "°"
elseif s.res_yaw then row.res = tostring(math.floor(V.norm(s.res_yaw) + 0.5)) .. "°" else row.res = "-" end row.hm = tostring(s.hits_n or 0) .. " / " .. tostring(s.miss_n or 0)
row.dist = (s.alive and s.dist) and tostring(math.floor(s.dist + 0.5)) or "-" row.hp = s.alive and tostring(math.floor(tonumber(s.hp) or 100)) or "dead" end end end V.pl_rn = n return alive_n end V.pl_alive = _v(0)
V.draw_players = function(now, acc, acc2) if not V.pl_on:get() or not V.res_on:get() then if V.pl_bg.Visible then V.pl_hide() end V.pl_drag = false return end local x0, y0 = V.pl_x:get(), V.pl_y:get() local rh = 16
local h = 44 + (math.max(1, V.pl_rn) * rh) local okp, mx, my = pcall(input.get_mouse_position) local okd, down = pcall(input.is_mouse_down, 1) down = okd and down == true
if V.ui_open and okp and type(mx) == "number" then local pressed = down and not V.pl_prev if pressed and mx >= x0 and mx <= (x0 + V.PL_W) and my >= y0 and my <= (y0 + 22) then V.pl_drag = true V.pl_dx = mx - x0
V.pl_dy = my - y0 end if V.pl_drag then if down then V.pl_x:set(math.floor(math.max(0, math.min(V.sw - V.PL_W, mx - V.pl_dx))), true) V.pl_y:set(math.floor(math.max(0, math.min(V.sh - h, my - V.pl_dy))), true)
x0 = V.pl_x:get() y0 = V.pl_y:get() else V.pl_drag = false end end else V.pl_drag = false end V.pl_prev = down if not V.pl_drag and (now - V.pl_t) < 0.12 then return end V.pl_t = now V.pl_alive = V.pl_collect()
h = 44 + (math.max(1, V.pl_rn) * rh) V.panel(V.pl_shadow, V.pl_bg, V.pl_line, x0, y0, V.PL_W, h, acc, acc2, V.pl_shine, V.pl_ic, "Players") V.text(V.pl_title, x0 + 30, y0 + 8, "PLAYERS", V.C_DIM)
V.text(V.pl_cnt, x0 + 30 + V.tw("PLAYERS", 11) + 6, y0 + 8, tostring(V.pl_alive) .. " alive", acc2) for c = 1, 6 do V.text(V.pl_head[c], x0 + V.PL_COLS[c], y0 + 24, V.PL_HEAD[c], V.C_DIM) end
V.rect(V.pl_hsep, x0 + 8, y0 + 37, V.PL_W - 16, 1, V.C_SEP) for r = 1, V.PL_N do local row = V.pl_rows[r] local cells = V.pl_cell[r] if r <= V.pl_rn and row then local ry = (y0 + 41) + ((r - 1) * rh) local s = row.s
local dead = not s.alive if row.tgt and V.pl_hl:get() and not dead then V.rect(V.pl_hl_r[r], x0 + 4, ry - 1, V.PL_W - 8, rh, V.mix(V.C_BG, acc, 0.22)) else V.hide(V.pl_hl_r[r]) end
local base = dead and V.C_DIM or V.C_TEXT local ncol = base if not dead and s.hard then ncol = V.C_BAD elseif not dead and row.tgt then ncol = acc end V.text(cells[1], x0 + V.PL_COLS[1], ry, row.name, ncol)
local ccol = (s.cls and not dead and V.res_cls:get()) and (V.CLS_COL[s.cls] or base) or V.C_DIM V.text(cells[2], x0 + V.PL_COLS[2], ry, row.cls, ccol)
V.text(cells[3], x0 + V.PL_COLS[3], ry, row.res, dead and V.C_DIM or acc2) local mcol = base if not dead and (s.misses or 0) >= 2 then mcol = V.C_WARN end V.text(cells[4], x0 + V.PL_COLS[4], ry, row.hm, mcol)
V.text(cells[5], x0 + V.PL_COLS[5], ry, row.dist, base) local hpn = tonumber(row.hp) local hcol = V.C_DIM if hpn then hcol = (hpn > 60) and V.C_GOOD or ((hpn > 25) and V.C_WARN or V.C_BAD) end
V.text(cells[6], x0 + V.PL_COLS[6], ry, row.hp, hcol) else V.hide(V.pl_hl_r[r]) for c = 1, 6 do V.hide(cells[c]) end end end if V.pl_rn == 0 then local cells = V.pl_cell[1]
V.text(cells[1], x0 + V.PL_COLS[1], y0 + 41, "no enemies", V.C_DIM) end end V.z_pos = _v(0) V.reapply_z = function() local n = math.min(V.REG_N, 80) for _ = 1, n do V.z_pos = (V.z_pos % V.REG_N) + 1
local d = V.REG[V.z_pos] local c = V.PC[d] local z = V.ZOF[d] if z and c and c.vis then V.setz(d, z) end end end V.z_timer = _v(0) client.set_event_callback("paint", function(dt) local pp = V.pf0()
local okm, em = pcall(V.overlay_paint, dt) if not okm then V.report("paint:overlay", em) end V.pf1("paint", pp) V.pf_report(V.now_s()) end) V.overlay_paint = function(dt) dt = dt or 0.016 V.z_timer = V.z_timer + dt
if V.z_timer > 0.25 then V.z_timer = 0; V.reapply_z() end local okf, ft0 = pcall(globals.frametime) local ft = (okf and type(ft0) == "number" and ft0 > 0) and ft0 or dt V.fps_n = V.fps_n + 1 local fnow = V.now_s()
if (fnow - V.fps_t0) >= 0.5 then local frames = V.rs_conn and V.rs_frames or V.fps_n local inst = frames / (fnow - V.fps_t0) if V.rs_conn and V.rs_dt > 0.2 and V.rs_frames > 5 then inst = V.rs_frames / V.rs_dt end
if not V.fps_src_logged then V.fps_src_logged = true V.log(150, 150, 170, "[NP] fps source: " .. ((V.rs_conn and V.rs_frames > 0) and "RenderStepped" or "paint")) end V.fps_s = (V.fps_s * 0.4) + (inst * 0.6) V.fps_n = 0
V.rs_frames = 0 V.rs_dt = 0 V.fps_t0 = fnow end V.tw_t = V.tw_t + dt if V.tw_t > 2 then V.tw_t = 0; V.kb_layout = nil end local now = V.now_s() local ACC = V.accent_col:get() or color(70, 160, 255)
local ACC2 = V.lighten(ACC, 0.45) if now - V.name_t > 2 then V.name_t = now local okl, lp = V.api("entity.get_local_player", entity.get_local_player) if okl and lp then
local _, inf = V.api("entity.get_info", entity.get_info, lp) if inf and inf.name then V.my_name = tostring(inf.name) end end local okp, lat = V.api("client.latency", client.latency)
if okp and type(lat) == "number" then V.ping_ms = math.floor(lat * 1000 + 0.5) end local okt, tstr = pcall(os.date, "%H:%M") if okt and type(tstr) == "string" then V.clock_s = tstr end end local el = V.intro_elapsed(now)
V.draw_intro(el, ACC, ACC2) local rel = V.reveal_t and (now - V.reveal_t) or -1 V.hide(V.hint_t) if el >= V.INTRO_LEN then local okh, eh = pcall(V.draw_hint, now, ACC, ACC2) if not okh then V.report("hint", eh) end end
local x0, y0 = V.wm_x:get(), V.wm_y:get() local pm = V.perf_mode:get() V.potato = (pm == "Potato") local panel_iv = (pm == "Potato") and 0.25 or ((pm == "Balanced") and 0.1 or (1 / 30))
local fx_iv = (pm == "Potato") and (1 / 15) or ((pm == "Balanced") and (1 / 30) or 0) local panels_due = V.dragging or V.panels_anim or (now - V.panels_t) > panel_iv if panels_due then local pdt = now - V.panels_t
if pdt > 0.1 then pdt = 0.1 end if pdt <= 0 then pdt = dt end V.panels_t = now V.panels_anim = false if rel >= -0.2 and rel < 1.0 then V.panels_anim = true end local cy = y0 local bx = x0 local v1, o1 = V.appear(rel, 0)
local v2, o2 = V.appear(rel, 0.12) local v3, o3 = V.appear(rel, 0.24) x0 = bx + o1 if V.wm_on:get() and v1 then local hh = 26 local title = V.wm_name:get() if title == nil or title == "" then title = "NeverPeek" end
local t1, t2 = title, "" if title == "NeverPeek" then t1 = "Never"; t2 = "Peek" end V.hide(V.wm_logo); V.hide(V.wm_logo_in); V.hide(V.wm_logo_t) V.hide(V.wm_logo_dot) local cx = x0 + 10 if t2 ~= "" then
V.text(V.wm_t1, cx, y0 + 5, t1, V.C_TEXT) V.text(V.wm_t2, cx + math.floor(V.tw_obj(V.wm_t1, t1, 14) + 0.5), y0 + 5, t2, ACC) else V.text(V.wm_t1, cx, y0 + 5, t1, V.C_TEXT) V.hide(V.wm_t2) end cx = cx + V.tw(t1 .. t2, 14)
local segs, sc, sn = {}, {}, 0 local fps = math.floor(V.fps_s + 0.5) sn = sn + 1; segs[sn] = tostring(fps) .. " FPS" sc[sn] = (fps < 30) and V.C_BAD or ((fps < 60) and V.C_WARN or V.C_TEXT) if V.ping_ms then
sn = sn + 1; segs[sn] = tostring(V.ping_ms) .. " MS" sc[sn] = (V.ping_ms > 200) and V.C_BAD or ((V.ping_ms > 120) and V.C_WARN or V.C_TEXT) end
if V.clock_s ~= "" then sn = sn + 1; segs[sn] = V.clock_s; sc[sn] = V.C_DIM end if V.my_name ~= "" then sn = sn + 1; segs[sn] = V.my_name; sc[sn] = ACC end for i = 1, V.SEG_N do if i <= sn then cx = cx + 9
V.rect(V.wm_sep[i], cx, y0 + 8, 1, 10, V.C_SEP) cx = cx + 9 V.text(V.wm_seg[i], cx, y0 + 6, segs[i], sc[i]) cx = cx + V.tw(segs[i], 13) else V.hide(V.wm_seg[i]); V.hide(V.wm_sep[i]) end end
local wm_target = (cx - x0) + 10 V.wm_w = V.approach(V.wm_w, wm_target, pdt, 14) if math.abs(wm_target - V.wm_w) > 0.5 then V.panels_anim = true end
V.panel(V.wm_shadow, V.wm_bg, V.wm_line, x0, y0, V.wm_w, hh, ACC, ACC2, V.wm_shine) V.handle_drag(x0, y0, V.wm_w, hh) cy = y0 + hh + 6 else V.hide(V.wm_shadow); V.hide(V.wm_bg); V.hide(V.wm_line)
V.hide(V.wm_logo); V.hide(V.wm_logo_t); V.hide(V.wm_t1); V.hide(V.wm_t2) V.hide(V.wm_logo_in); V.hide(V.wm_logo_dot); V.hide(V.wm_shine) V.hide_range(V.wm_seg, V.SEG_N); V.hide_range(V.wm_sep, V.SEG_N) end x0 = bx + o2
if V.wm_stats:get() and v2 then local total = V.stats.hits + V.stats.misses local hitp = V.pct(V.stats.hits, total) local pcol = (hitp >= 60) and V.C_GOOD or ((hitp >= 40) and V.C_WARN or V.C_BAD)
local kd = V.stats.deaths > 0 and (V.stats.kills / V.stats.deaths) or V.stats.kills local kd_s = tostring(math.floor(kd * 100 + 0.5) / 100)
local v1 = tostring(V.stats.hits) .. " hit  ·  " .. tostring(V.stats.misses) .. " miss" local v2 = "" if V.hp_src then
v2 = tostring(V.stats.taken) .. " hit  ·  " .. tostring(V.stats.taken_hs) .. " hs  ·  " .. tostring(V.stats.deaths) .. " deaths" elseif V.armor_live then
v2 = tostring(V.stats.taken) .. " hit  ·  " .. tostring(V.stats.deaths) .. " deaths" else v2 = tostring(V.stats.deaths) .. " deaths  ·  hits n/a" end
local vm = "k/d " .. kd_s .. "  ·  streak " .. tostring(V.stats.streak) .. "  ·  best " .. tostring(V.stats.best_streak) local bd = V.miss_bd:get()
local v3 = "res " .. tostring(V.stats.res_miss) .. "  ·  spr " .. tostring(V.stats.spread) .. "  ·  occ " .. tostring(V.stats.occ) .. "  ·  ? " .. tostring(V.stats.other) .. "  ·  dead " .. tostring(V.stats.dead)
local key_w = V.tw("ENEMY", 12) + 10 local target = math.max(V.wm_w, key_w + V.tw(v1, 12) + 20, key_w + V.tw(v2, 12) + 20, V.tw(vm, 11) + 20, 200) if bd then target = math.max(target, key_w + V.tw(v3, 12) + 20) end
V.st_w = V.approach(V.st_w, target, pdt, 14) if math.abs(target - V.st_w) > 0.5 then V.panels_anim = true end local h = bd and 110 or 86
V.panel(V.st_shadow, V.st_bg, V.st_line, x0, cy, V.st_w, h, ACC, ACC2, V.st_shine, V.st_ic, "Rage") local lx = x0 + 10 V.text(V.st_title, lx + 20, cy + 8, "HIT RATE", V.C_DIM)
V.text(V.st_pct, (lx + 20) + V.tw("HIT RATE", 11) + 6, cy + 8, tostring(hitp) .. "%", pcol) local bw = V.st_w - 20 V.bar_k = V.approach(V.bar_k, hitp / 100, pdt, 6)
if math.abs(hitp / 100 - V.bar_k) > 0.002 then V.panels_anim = true end V.rect(V.st_barbg, lx, cy + 24, bw, 4, V.C_BG2) if V.bar_k > 0.005 then V.rect(V.st_bar, lx, cy + 24, bw * V.bar_k, 4, ACC)
V.grad(V.st_bar, ACC, ACC2) else V.hide(V.st_bar) end V.text(V.st_k1, lx, cy + 34, "YOU", V.C_DIM) V.text(V.st_v1, lx + key_w, cy + 34, v1, V.C_TEXT) V.text(V.st_k2, lx, cy + 50, "ENEMY", V.C_DIM)
V.text(V.st_v2, lx + key_w, cy + 50, v2, (V.hp_src or V.armor_live) and V.C_TEXT or V.C_DIM) if bd then V.text(V.st_k3, lx, cy + 66, "MISS", V.C_DIM) V.text(V.st_v3, lx + key_w, cy + 66, v3, V.C_TEXT)
local parts_n = { V.stats.res_miss, V.stats.spread, V.stats.occ, V.stats.other, V.stats.dead } local all = 0 for i = 1, 5 do all = all + parts_n[i] end if (all - V.stats.dead) <= 0 then all = 0 end
V.rect(V.st_segbg, lx, cy + 83, bw, 4, V.C_BG2) local sx = lx for i = 1, 5 do local wseg = (all > 0) and ((bw * parts_n[i]) / all) or 0 if wseg >= 1 then V.rect(V.st_seg[i], sx, cy + 83, wseg, 4, V.SEG_COL[i])
sx = sx + wseg else V.hide(V.st_seg[i]) end end V.text(V.st_misc, lx, cy + 92, vm, V.C_DIM) else V.hide(V.st_k3); V.hide(V.st_v3); V.hide(V.st_segbg); V.hide_range(V.st_seg, 5) V.text(V.st_misc, lx, cy + 67, vm, V.C_DIM)
end cy = cy + h + 6 else V.hide(V.st_shadow); V.hide(V.st_bg); V.hide(V.st_line); V.hide(V.st_shine); V.icon_hide(V.st_ic) V.hide(V.st_title); V.hide(V.st_pct); V.hide(V.st_barbg); V.hide(V.st_bar)
V.hide(V.st_k1); V.hide(V.st_v1); V.hide(V.st_k2); V.hide(V.st_v2); V.hide(V.st_misc) V.hide(V.st_k3); V.hide(V.st_v3); V.hide(V.st_segbg); V.hide_range(V.st_seg, 5) end x0 = bx + o3 if V.show_binds:get() and v3 then
local rows_n, rn, rs, rc = 0, {}, {}, {} local function row(n, st_, c) rows_n = rows_n + 1; rn[rows_n], rs[rows_n], rc[rows_n] = n, st_, c end local aa_en = V.aa_on:get()
row("anti-aim  [" .. V.k_aa:get() .. "]", aa_en and "on" or "off", aa_en and V.C_GOOD or V.C_BAD) row("pitch  [" .. V.k_pitch:get() .. "]", V.pitch_up and "up" or "down", V.C_TEXT)
row("inverter  [" .. V.k_inv:get() .. "]", (V.cur_inv ~= V.brute_inv) and "right" or "left", V.C_TEXT)
if aa_en then row("desync", tostring(V.lim_shown_l) .. " / " .. tostring(V.lim_shown_r), (V.lim_jit:get() ~= "Off") and ACC or V.C_TEXT) else row("desync", "off", V.C_DIM) end
if V.hf_on:get() then row("hard flick", tostring(V.hf_cur) .. "°", ACC) else row("hard flick", "off", V.C_DIM) end if not V.def_on:get() then row("defensive", "off", V.C_DIM)
elseif aa_en and V.now_tick() < V.peek_until then row("defensive", "peek", V.C_BAD) elseif aa_en and V.now_tick() < V.flick_until then row("defensive", "flick", V.C_WARN) else row("defensive", "idle", V.C_DIM) end
if not V.acc_shoot:get() then row("accurate shot", "off", V.C_DIM) elseif V.last_shooting then row("accurate shot", "active", ACC) else row("accurate shot", "idle", V.C_DIM) end
if not V.scope_wait:get() then row("wait scope", "off", V.C_DIM) elseif V.scope_blocked then row("wait scope", "waiting", V.C_WARN) else row("wait scope", "ready", V.C_DIM) end
if not V.res_on:get() then row("resolver", "off", V.C_DIM) elseif V.res_mode:get() == "Lua Brute" then row("resolver", "lua brute", ACC)
elseif V.bi_cycle:get() ~= "Off" then row("resolver", string.lower(V.BI_MODES[V.bi_idx]), V.C_TEXT) else row("resolver", "built-in", V.C_TEXT) end
if V.hc_applied then row("hitchance", tostring(V.hc_applied) .. "%", V.C_TEXT) else row("hitchance", "menu", V.C_DIM) end if not V.kb_layout then local name_w = 0 for i = 1, rows_n do local a = V.tw(rn[i], 12)
if a > name_w then name_w = a end end local state_w = math.max(V.tw("lua brute", 12), V.tw("waiting", 12), V.tw("86 / 86", 12)) V.kb_layout = { col = 22 + name_w + 20, w = 22 + name_w + 20 + state_w + 20 } end
local col_x = V.kb_layout.col V.kb_w = V.kb_layout.w local h = 26 + V.KB_N * 16 V.panel(V.kb_shadow, V.kb_bg, V.kb_line, x0, cy, V.kb_w, h, ACC, ACC2, V.kb_shine, V.kb_ic, "Binds")
V.text(V.kb_title, x0 + 30, cy + 8, "KEYBINDS", V.C_DIM) for i = 1, V.KB_N do if i <= rows_n then local ry = cy + 24 + ((i - 1) * 16) local dim = rc[i] == V.C_DIM local plain = rc[i] == V.C_TEXT
V.text(V.kb_name[i], x0 + 22, ry, rn[i], dim and V.C_DIM or V.C_TEXT) local vw = V.tw(rs[i], 12) local vx = ((x0 + V.kb_w) - 12) - vw V.text(V.kb_state[i], vx, ry, rs[i], rc[i]) local dc = V.C_SEP
if not dim then dc = plain and ACC or rc[i] end V.rect(V.kb_dot[i], x0 + 11, ry + 4, 6, 6, dc) if dim then V.hide(V.kb_pill[i]) else
V.rect(V.kb_pill[i], vx - 6, ry - 1, vw + 12, 15, plain and V.KB_PLAIN or V.mix(V.C_BG, rc[i], 0.24)) end else V.hide(V.kb_name[i]); V.hide(V.kb_state[i]); V.hide(V.kb_pill[i]); V.hide(V.kb_dot[i]) end end else
V.hide(V.kb_shadow); V.hide(V.kb_bg); V.hide(V.kb_line); V.hide(V.kb_title); V.hide(V.kb_shine); V.icon_hide(V.kb_ic) V.hide_range(V.kb_pill, V.KB_N); V.hide_range(V.kb_dot, V.KB_N)
V.hide_range(V.kb_name, V.KB_N); V.hide_range(V.kb_state, V.KB_N) end end local scx, scy = V.sw / 2, V.sh / 2 local fx_due = (now - V.fx_t) >= fx_iv if fx_due then V.fx_t = now end
local cross_culled = V.cull_scope:get() and V.is_scoped_now if not fx_due then elseif V.show_cross:get() and rel >= 0.36 and not cross_culled then local lt, lc = {}, {}
local function slot(i, t, c) lt[i], lc[i] = t, c end local pulse = (math.sin(now * 3) + 1) / 2 slot(1, string.upper(V.wm_name:get() ~= "" and V.wm_name:get() or "NeverPeek"), pulse > 0.5 and ACC or ACC2)
if V.aa_on:get() then slot(2, (V.pitch_up and "UP" or "DOWN") .. "  ·  " .. ((V.cur_inv ~= V.brute_inv) and "RIGHT" or "LEFT"), V.C_TEXT) else slot(2, "AA OFF", V.C_BAD) end if now - V.dt_t > 0.1 then V.dt_t = now
local okd, ch = V.api("rage.is_double_tap_charged", rage.is_double_tap_charged) V.dt_ok = okd V.dt_charged = ch end local okdt, charged = V.dt_ok, V.dt_charged
if okdt and charged ~= nil then slot(3, "DT", charged and V.C_GOOD or V.C_DIM) end if V.def_on:get() then if V.aa_on:get() and V.now_tick() < V.flick_until then slot(4, "DEFENSIVE", V.C_WARN) else
slot(4, "DEFENSIVE", V.C_DIM) end end if V.show_state:get() then local sc = V.C_GOOD if V.last_state == "moving" then sc = V.C_WARN elseif V.last_state == "in air" or V.last_state == "air crouch" then sc = V.C_BAD end
slot(5, "~ " .. V.last_state, sc) end local tgt = V.cur_tgt if tgt and tgt.player then local s = V.st(tgt.player) if s and V.hard_on:get() and s.hard then slot(6, "BAIM  ·  HARD", V.C_BAD) elseif V.want_baim(s) then
slot(6, "BAIM", V.C_BAD) end if s and s.cls and V.res_cls:get() then slot(7, "vs " .. V.cls_label(s), V.C_DIM) end end for i = 1, V.CI_N do if lt[i] then V.text(V.ci_txt[i], scx, scy + 22 + (i - 1) * 13, lt[i], lc[i])
else V.hide(V.ci_txt[i]) end end if V.aa_on:get() then local right = (V.cur_inv ~= V.brute_inv) V.text(V.ci_arrow_l, scx - 48, scy - 11, "<", right and V.C_SEP or ACC)
V.text(V.ci_arrow_r, scx + 48, scy - 11, ">", right and ACC or V.C_SEP) else V.hide(V.ci_arrow_l); V.hide(V.ci_arrow_r) end else V.hide_range(V.ci_txt, V.CI_N) V.hide(V.ci_arrow_l); V.hide(V.ci_arrow_r) end
local tmax = V.cull_toast:get() and 3 or V.TN local p2 = V.pf0() if V.show_toasts:get() then V.draw_toasts(now, scx, scy, tmax, math.min(0.1, math.max(0.001, dt or 0.016))) else for i = 1, V.TN do V.hide_toast(i) end end
V.pf1("toasts", p2) local p3 = V.pf0() if el >= V.INTRO_LEN then V.draw_announcer(now, ACC, ACC2) end V.pf1("announcer", p3) if rel >= 0.3 then local p4 = V.pf0() V.draw_labels(now, ACC) V.pf1("labels", p4)
local p5 = V.pf0() V.draw_players(now, ACC, ACC2) V.pf1("players", p5) local p6 = V.pf0() V.draw_dtbar(now, ACC, ACC2) V.pf1("dtbar", p6) end end V.dt_shadow = V.mk_rect(6, V.Z_SHADOW) V.dt_bg = V.mk_rect(5, V.Z_BG)
V.dt_line = V.mk_rect(1, V.Z_DECO) V.dt_shine = V.mk_rect(1, V.Z_DECO + 1) V.dt_ic = V.mk_icon() V.dt_title = V.mk_text(11, false, false) V.dt_state = V.mk_text(11, false, false) V.dt_tag = V.mk_text(10, false, false)
V.dt_track = V.mk_rect(2, V.Z_DECO) V.dt_fill = V.mk_rect(2, V.Z_DECO + 1) V.dt_ticks = V.mk_text(10, false, false) V.DTB = { w = 180, h = 46, drag = false, dx = 0, dy = 0, prev = false, t = -10, fill = 0 }
V.dt_hide = function() V.hide(V.dt_shadow); V.hide(V.dt_bg); V.hide(V.dt_line); V.hide(V.dt_shine); V.icon_hide(V.dt_ic)
V.hide(V.dt_title); V.hide(V.dt_state); V.hide(V.dt_tag); V.hide(V.dt_track); V.hide(V.dt_fill); V.hide(V.dt_ticks) end V.draw_dtbar = function(now, acc, acc2) if not V.dt_bar:get() then
if V.dt_bg.Visible then V.dt_hide() end V.DTB.drag = false return end local B = V.DTB local x0 = V.dt_x:get() local y0 = V.dt_y:get() local okp, mx, my = pcall(input.get_mouse_position)
local okd, down = pcall(input.is_mouse_down, 1) down = okd and down == true if V.ui_open and okp and type(mx) == "number" then local pressed = down and not B.prev
if pressed and mx >= x0 and mx <= (x0 + B.w) and my >= y0 and my <= (y0 + B.h) then B.drag = true B.dx = mx - x0 B.dy = my - y0 end if B.drag then if down then
V.dt_x:set(math.floor(math.max(0, math.min(V.sw - B.w, mx - B.dx))), true) V.dt_y:set(math.floor(math.max(0, math.min(V.sh - B.h, my - B.dy))), true) x0 = V.dt_x:get() y0 = V.dt_y:get() else B.drag = false end end else
B.drag = false end B.prev = down if not B.drag and (now - B.t) < 0.03 then return end local fdt = math.min(0.1, now - B.t) B.t = now V.dt_read(now) if V.smart_dt:get() then V.smart_hold() else V.DT.hold = "" end
local D = V.DT local k = D.charge / 14 B.fill = B.fill + ((k - B.fill) * math.min(1, fdt * 14)) V.panel(V.dt_shadow, V.dt_bg, V.dt_line, x0, y0, B.w, B.h, acc, acc2, V.dt_shine, V.dt_ic, "Hard Flick")
V.text(V.dt_title, x0 + 30, y0 + 8, "DOUBLE TAP", V.C_DIM) local st = "OFF" local sc = V.C_DIM if not D.ok then st = "N/A" elseif D.on and D.charge >= 14 then st = "READY"; sc = V.C_GOOD
elseif D.on then st = "CHARGING"; sc = V.C_WARN end if D.on and D.hold ~= "" and D.charge >= 14 then st = string.upper(D.hold); sc = acc2 end V.text(V.dt_state, ((x0 + B.w) - 10) - V.tw(st, 11), y0 + 8, st, sc)
local bx = x0 + 10 local bw = B.w - 20 V.rect(V.dt_track, bx, y0 + 27, bw, 6, V.C_SEP) local fw = math.floor(bw * math.max(0, math.min(1, B.fill))) if fw >= 2 then
V.rect(V.dt_fill, bx, y0 + 27, fw, 6, D.on and acc or V.C_DIM) V.grad(V.dt_fill, D.on and acc or V.C_DIM, D.on and acc2 or V.C_DIM) else V.hide(V.dt_fill) end local tk = tostring(math.floor(D.charge + 0.5)) .. "/14"
V.text(V.dt_ticks, bx, y0 + 35, tk, V.C_DIM) if V.lethal_now then local lt = (V.lethal_now == "dt") and "LETHAL · DT" or "LETHAL" V.text(V.dt_tag, ((x0 + B.w) - 10) - V.tw(lt, 10), y0 + 35, lt, V.C_BAD) else
V.hide(V.dt_tag) end end V.BEE = { MAX = 12, TR = 3, list = {}, mode = "home", until_t = 0, ax = 0, ay = 0, az = 0, hs = false, t = 0, shown = false, honey_t = -10, hx = 0, hy = 0, hz = 0 }
V.rnd01 = function() return math.random(0, 1000) / 1000 end V.BEE_YEL = color(255, 205, 40) V.BEE_BLK = color(25, 20, 10) V.BEE_WNG = color(220, 240, 255) V.BEE_POL = color(255, 225, 90) V.bee_make = function(i)
local b = { emo = V.mk_text(20, false, true, V.Z_CROSS + 2), w1 = V.mk_rect(5, V.Z_CROSS), w2 = V.mk_rect(5, V.Z_CROSS),
body = V.mk_rect(6, V.Z_CROSS + 1), s1 = V.mk_rect(0, V.Z_CROSS + 2), s2 = V.mk_rect(0, V.Z_CROSS + 2), eye = V.mk_rect(2, V.Z_CROSS + 2), tr = {}, hist = {}, ph = (i * 2.399) + (V.rnd01() * 0.5),
sp = 0.8 + (V.rnd01() * 0.45), rr = 0.8 + (V.rnd01() * 0.4), hh = V.rnd01() * 6.28, k = 2.6 + (V.rnd01() * 2.2), dir = (V.rnd01() < 0.15) and -1 or 1, has = false, x = 0, y = 0, z = 0, px = false, sz = -1, }
b.tr[1] = V.mk_rect(2, V.Z_CROSS) b.tr[2] = V.mk_rect(2, V.Z_CROSS) b.tr[3] = V.mk_rect(2, V.Z_CROSS) return b end for i = 1, V.BEE.MAX do V.BEE.list[i] = V.bee_make(i) end
V.bee_honey = V.mk_text(24, false, true, V.Z_CROSS + 3) V.bee_hide = function(b) V.hide(b.emo); V.hide(b.w1); V.hide(b.w2); V.hide(b.body); V.hide(b.s1); V.hide(b.s2); V.hide(b.eye)
V.hide(b.tr[1]); V.hide(b.tr[2]); V.hide(b.tr[3]) end V.bees_hide_all = function() for i = 1, V.BEE.MAX do local b = V.BEE.list[i] V.bee_hide(b) b.has = false b.hist = {} end V.hide(V.bee_honey) V.BEE.shown = false end
V.bee_root = function() local ch = game:GetService("Players").LocalPlayer.Character if not ch then return false end
local r = ch:FindFirstChild("HumanoidRootPart") or ch:FindFirstChild("Torso") or ch:FindFirstChild("UpperTorso") or ch.PrimaryPart if not r then return false end return r.Position end
V.bee_cam = function() return workspace.CurrentCamera.CFrame.Position end V.bee_proj = function(x, y, z) local ok, sx, sy, on = pcall(render.world_to_screen, Vector3.new(x, y, z))
if ok and on and type(sx) == "number" then return sx, sy end return false, false end V.bees_kill = function(target, hs) if not V.bee_on:get() or not V.bee_attack:get() then return end local s = V.st(target)
if not s or not s.px then return end local B = V.BEE B.mode = "attack" B.ax = s.px B.ay = s.py B.az = s.pz B.hs = hs and true or false local now = V.now_s() B.until_t = now + (hs and 4 or 3) B.honey_t = now B.hx = s.px
B.hy = s.py B.hz = s.pz end V.bee_sq = function(d, x, y, w, h, col, a) V.rect(d, x, y, math.max(1, w), math.max(1, h), col) V.alpha(d, a) end V.bee_drawn = function(b, x, y, s, a, t, face) local bw = s
local bh = s * 0.62 local bx = x - (bw / 2) local by = y - (bh / 2) local flap = 0.55 + (0.45 * math.abs(math.sin((t * 38) + b.ph))) local ww = s * 0.42 local wh = s * 0.5 * flap
V.bee_sq(b.w1, x - (ww * 0.9), (by - wh) + (bh * 0.25), ww, wh, V.BEE_WNG, a * 0.75) V.bee_sq(b.w2, x + (ww * 0.05), (by - (wh * 0.85)) + (bh * 0.25), ww, wh * 0.9, V.BEE_WNG, a * 0.6)
V.bee_sq(b.body, bx, by, bw, bh, V.BEE_YEL, a) local sw = math.max(1, s * 0.12) V.bee_sq(b.s1, (x - (bw * 0.12)) - (sw / 2), by + 1, sw, bh - 2, V.BEE_BLK, a)
V.bee_sq(b.s2, (x + (bw * 0.14)) - (sw / 2), by + 1, sw, bh - 2, V.BEE_BLK, a) local es = math.max(2, s * 0.14) local ex = (face > 0) and ((bx + bw) - (es * 1.9)) or (bx + (es * 0.9))
V.bee_sq(b.eye, ex, by + (bh * 0.25), es, es, V.BEE_BLK, a) V.hide(b.emo) end V.bee_emoji = function(b, x, y, s, a) local fs = math.floor(s) if b.sz ~= fs then b.sz = fs b.emo.Size = fs end
V.text(b.emo, x, y - (s / 2), "🐝", V.C_WHITE) V.alpha(b.emo, a) V.hide(b.w1); V.hide(b.w2); V.hide(b.body); V.hide(b.s1); V.hide(b.s2); V.hide(b.eye) end V.bee_trail_draw = function(b, a) local on = V.bee_trail:get()
for k = 1, V.BEE.TR do local d = b.tr[k] local p = b.hist[k] if on and p then local sz = math.max(1, 4 - (k * 0.8)) V.bee_sq(d, p.x - (sz / 2), p.y - (sz / 2), sz, sz, V.BEE_POL, a * (1 - (k / (V.BEE.TR + 1)))) else
V.hide(d) end end end V.bee_update = function(b, i, n, now, dt, home, cpos, rootd, fp) local B = V.BEE local attack = B.mode == "attack" local cx = home.X local cy = home.Y local cz = home.Z
local R = V.bee_rad:get() * b.rr local hy = 1 + (0.6 * math.sin((now * 1.7) + b.hh)) local spd = 1.6 if attack then cx = B.ax cy = B.ay cz = B.az R = 1.2 + (0.6 * b.rr)
hy = 0.6 + (1.4 * math.abs(math.sin((now * 2.3) + b.hh))) spd = 3.2 end local ang = (b.dir * now * spd * b.sp) + b.ph + ((i - 1) * ((math.pi * 2) / n)) R = R * (0.85 + (0.15 * math.sin((now * 0.9) + b.hh)))
local tx = cx + (math.cos(ang) * R) + (0.12 * math.sin((now * 23) + (i * 3.1))) local ty = cy + hy + (0.1 * math.sin((now * 29) + (i * 1.3)))
local tz = cz + (math.sin(ang) * R) + (0.12 * math.cos((now * 19) + (i * 2.2))) if not b.has then b.x = tx b.y = ty b.z = tz b.has = true else local k = math.min(1, dt * b.k) b.x = b.x + ((tx - b.x) * k)
b.y = b.y + ((ty - b.y) * k) b.z = b.z + ((tz - b.z) * k) end local dx = b.x - cpos.X local dy = b.y - cpos.Y local dz = b.z - cpos.Z local d = math.sqrt((dx * dx) + (dy * dy) + (dz * dz))
if (fp and not attack) or d < 1.2 then V.bee_hide(b) b.hist = {} return end local x, y = V.bee_proj(b.x, b.y, b.z) if not x then V.bee_hide(b) b.hist = {} return end
local s = math.max(6, math.min(64, V.bee_size:get() * (12 / math.max(2, d)))) local a = (d > rootd and not attack) and 0.6 or 1 local face = 1 if b.px then face = (x >= b.px) and 1 or -1 end b.px = x
if not b.ht or (now - b.ht) >= 0.035 then b.ht = now b.hist[3] = b.hist[2] b.hist[2] = b.hist[1] b.hist[1] = { x = x, y = y } end V.bee_trail_draw(b, a) if V.bee_style:get() == "Drawn" then
V.bee_drawn(b, x, y, s, a, now, face) else V.bee_emoji(b, x, y, s, a) end end V.bee_draw_honey = function(now) local B = V.BEE local age = now - B.honey_t if age > 1.6 then V.hide(V.bee_honey) return end
local x, y = V.bee_proj(B.hx, B.hy + 2 + (age * 1.2), B.hz) if not x then V.hide(V.bee_honey) return end local pop = math.min(1, age / 0.18) local fade = 1 - math.max(0, (age - 1.1) / 0.5)
local fs = math.floor(14 + (14 * pop)) if V.bee_honey_sz ~= fs then V.bee_honey_sz = fs V.bee_honey.Size = fs end V.text(V.bee_honey, x, y - (fs / 2), B.hs and "👑" or "🍯", V.C_WHITE)
V.alpha(V.bee_honey, math.max(0, fade)) end V.bees_frame = function() if not V.bee_on:get() then if V.BEE.shown then V.bees_hide_all() end return end local okr, home = pcall(V.bee_root) if not okr or not home then
if V.BEE.shown then V.bees_hide_all() end return end local okc, cpos = pcall(V.bee_cam) if not okc or not cpos then return end local B = V.BEE local now = V.now_s() local dt = math.max(0, math.min(0.1, now - B.t))
B.t = now if B.mode == "attack" and now > B.until_t then B.mode = "home" end local rdx = home.X - cpos.X local rdy = home.Y - cpos.Y local rdz = home.Z - cpos.Z
local rootd = math.sqrt((rdx * rdx) + (rdy * rdy) + (rdz * rdz)) local fp = rootd < 1.5 local n = math.floor(V.bee_n:get()) for i = 1, B.MAX do local b = B.list[i] if i <= n then
local ok, err = pcall(V.bee_update, b, i, n, now, dt, home, cpos, rootd, fp) if not ok then V.bee_hide(b) if not B.err then B.err = true V.log(255, 120, 120, "[NP] bees: " .. tostring(err)) end end else V.bee_hide(b)
b.has = false end end V.bee_draw_honey(now) B.shown = true end client.set_event_callback("paint", function() local pb = V.pf0() local ok, err = pcall(V.bees_frame) V.pf1("bees", pb) if not ok and not V.BEE.err2 then
V.BEE.err2 = true V.log(255, 120, 120, "[NP] bees: " .. tostring(err)) end end) V.MW = _v(820) V.MSIDE = _v(196) V.MTOP = _v(54) V.MPAD = _v(16) V.ROWH = _v(32) V.CPAD = _v(6) V.GH = _v(22)
V.MCW = (V.MW - V.MSIDE) - (V.MPAD * 2) V.MCOLW = math.floor((V.MCW - 12) / 2) V.M_SLOTS = _v(30) V.M_DD = _v(8) V.SIDE_GROUPS = { { "ANTI-AIM", { "Anti-Aim", "AA States", "Defensive", "Hard Flick" } },
{ "RAGEBOT", { "Resolver", "Rage" } }, { "COMMON", { "Visuals", "Players", "Extras", "Grenades" } }, { "SETTINGS", { "Binds", "Config" } }, } V.TAB_GROUPS = { ["Anti-Aim"] = { "YAW & LIMITS", "INVERTER & BRUTE" },
["AA States"] = { "STATE", "OVERRIDE" }, ["Defensive"] = { "FLICKS", "SHOOTING & PEEK" }, ["Hard Flick"] = { "SPEED", "ANGLES" }, ["Resolver"] = { "MODE", "TARGETING" }, ["Rage"] = { "ACCURACY", "FAKE LAG & SAFETY" },
["Visuals"] = { "INDICATORS", "STYLE" }, ["Players"] = { "LIST", "POSITION" }, ["Extras"] = { "ANNOUNCER", "KILL SAY" }, ["Grenades"] = { "PREDICTION", "LIVE & DAMAGE" }, ["Binds"] = { "KEYS", "MORE" },
["Config"] = { "SLOTS", "CODE & PRESETS" }, } V.m_item_h = function(w) return V.ROWH end V.m_layout = function(sec) if sec.lay then return sec.lay end local vis, vn = {}, 0 for i = 1, sec.n do local w = sec.items[i]
if not w.hidden then vn = vn + 1; vis[vn] = w end end local half = math.ceil(vn / 2) local lay = {} for i = 1, vn do if i <= half then lay[i] = { col = 0, y = (i - 1) * V.ROWH }
else lay[i] = { col = 1, y = ((i - half) - 1) * V.ROWH } end end sec.lay = lay sec.vis = vis sec.vn = vn sec.rows0 = half sec.rows1 = vn - half sec.hmax = math.max(half, vn - half) * V.ROWH return lay end V.TAB_IDX = {}
for i = 1, V.UI_TN do V.TAB_IDX[V.UI_TABS[i].name] = i end V.SIDE_ROWS = {} V.SIDE_N = _v(0) do local yy = 72 for g = 1, #V.SIDE_GROUPS do local grp = V.SIDE_GROUPS[g] V.SIDE_N = V.SIDE_N + 1
V.SIDE_ROWS[V.SIDE_N] = { kind = "head", text = grp[1], y = yy, g = g } yy = yy + 22 for j = 1, #grp[2] do local ti = V.TAB_IDX[grp[2][j]] if ti then V.SIDE_N = V.SIDE_N + 1
V.SIDE_ROWS[V.SIDE_N] = { kind = "tab", tab = ti, y = yy, g = g } yy = yy + 32 end end yy = yy + 8 end V.SIDE_END = yy end V.m_tab_y = function(i) for r = 1, V.SIDE_N do local row = V.SIDE_ROWS[r]
if row.kind == "tab" and row.tab == i then return (row.dy or row.y) + 15 end end return 0 end V.MH = math.max(520, V.SIDE_END + 70) for i = 1, V.UI_TN do V.m_layout(V.UI_TABS[i])
V.MH = math.max(V.MH, (((V.MTOP + V.GH) + V.UI_TABS[i].hmax) + (V.CPAD * 2)) + V.MPAD) end V.MC = { win = color(15, 17, 23), side = color(12, 14, 19), card = color(20, 23, 30), cardb = color(33, 37, 47),
row = color(27, 31, 40), sel = color(31, 36, 47), hov = color(24, 28, 36), chip = color(23, 27, 35), box = color(29, 33, 43), off = color(48, 53, 66), dim = color(118, 126, 145), text = color(214, 220, 232),
head = color(98, 106, 124), shadow = color(0, 0, 0), } V.Z0 = _v(50) V.MK = function(r, z) return V.mk_rect(r, V.Z0 + z) end V.MT = function(sz, z, c, b) return V.mk_text(sz, false, c and true or false, V.Z0 + z, b) end
V.m_shadow = V.MK(14, -1) V.m_win = V.MK(12, 0) V.m_side = V.MK(12, 1) V.m_sidefix = V.MK(0, 1) V.m_sideln = V.MK(0, 2) V.m_logo_glow = V.MK(12, 2) V.m_logo = V.MK(9, 3) V.m_logo_in = V.MK(7, 4)
V.m_logo_t = V.MT(16, 6, true, true) V.m_logo_dot = V.MK(3, 7) V.m_title_a = V.MT(15, 5, false, true) V.m_title = V.MT(15, 6, false, true) V.m_sub = V.MT(11, 5) V.m_chip1 = V.MK(8, 3) V.m_chip1_i = V.MK(3, 4)
V.m_chip1_t = V.MT(13, 6) V.m_chip1_a = V.MT(11, 6) V.m_chip2 = V.MK(8, 3) V.m_chip2_t = V.MT(12, 6) V.m_gh = { V.MT(10, 5), V.MT(10, 5) } V.m_card = { V.MK(10, 2), V.MK(10, 2) } V.m_ubg = V.MK(10, 2)
V.m_uav = V.MK(15, 3) V.m_uav_t = V.MT(13, 6, true) V.m_un = V.MT(13, 5) V.m_us = V.MT(10, 5) V.m_uch = V.MT(13, 5) V.m_sh = {} V.m_sbg = {} V.m_sic = {} V.m_stx = {} V.m_sarr = {} V.G_OPEN = {} V.G_K = {}
for g = 1, #V.SIDE_GROUPS do V.G_OPEN[g] = true V.G_K[g] = 1 end V.icon_alpha = function(ic, a) if ic.ka == a then return end ic.ka = a
V.alpha(ic[1], a); V.alpha(ic[2], a); V.alpha(ic[3], a); V.alpha(ic[4], a); V.alpha(ic[5], a); V.alpha(ic[6], a) end for r = 1, V.SIDE_N do V.m_sarr[r] = V.MT(12, 5) V.m_sh[r] = V.MT(10, 5) V.m_sbg[r] = V.MK(7, 2)
V.m_sic[r] = { V.MK(2, 4), V.MK(2, 4), V.MK(2, 4), V.MK(2, 4), V.MK(2, 4), V.MK(2, 4) } V.m_stx[r] = V.MT(13, 5) end V.m_slot = {} for i = 1, V.M_SLOTS do
V.m_slot[i] = { bg = V.MK(6, 3), lb = V.MT(13, 5), a = V.MK(10, 4), b = V.MK(7, 5), c = V.MK(6, 6), p = V.MK(6, 4), v = V.MT(12, 6), bx = V.MK(6, 4), bt = V.MT(12, 5), ar = V.MT(12, 5) } end V.m_ddshadow = V.MK(10, 9)
V.m_ddbg = V.MK(8, 10) V.m_ddhl = V.MK(6, 11) V.m_ddtx = {} for i = 1, V.M_DD do V.m_ddtx[i] = V.MT(12, 12) end V.m_cursor = V.MK(3, 20) V.mk_tri = function(z) local d = Drawing.new("Triangle") d.Visible = false
pcall(function() d.Filled = true end) pcall(function() d.Thickness = 1 end) V.ZOF[d] = z V.setz(d, z) return V.reg(d) end V.mk_circ = function(z, filled) local d = Drawing.new("Circle") d.Visible = false
pcall(function() d.Filled = filled and true or false end) pcall(function() d.Thickness = 1.5 end) pcall(function() d.NumSides = 32 end) V.ZOF[d] = z V.setz(d, z) return V.reg(d) end
V.CUR = { trail = {}, tn = 0, rip_t = -10, rip_x = 0, rip_y = 0, press_t = -10, hot = 0, spin = 0 } V.cur_glow = V.mk_circ(V.Z0 + 21, true) V.cur_rip = V.mk_circ(V.Z0 + 21, false) V.cur_rip2 = V.mk_circ(V.Z0 + 21, false)
V.cur_trail = {} for i = 1, 6 do V.cur_trail[i] = V.mk_circ(V.Z0 + 21, true) end V.CUR_SL = 10 V.cur_so = {} V.cur_sf = {} for i = 1, V.CUR_SL do V.cur_so[i] = V.mk_rect(0, V.Z0 + 22)
V.cur_sf[i] = V.mk_rect(0, V.Z0 + 23) end V.cur_shine = V.mk_rect(1, V.Z0 + 24) V.cur_dot = V.mk_circ(V.Z0 + 24, true) V.CUR_OBJS = { V.cur_glow, V.cur_rip, V.cur_rip2, V.cur_shine, V.cur_dot } V.cur_hide = function()
for _, d in ipairs(V.CUR_OBJS) do V.hide(d) end for i = 1, 6 do V.hide(V.cur_trail[i]) end for i = 1, V.CUR_SL do V.hide(V.cur_so[i]); V.hide(V.cur_sf[i]) end end V.arrow_w = function(yy, L, W)
if yy <= W then return yy end return W - (((yy - W) * W) / (L - W)) end V.tri = function(d, ax, ay, bx, by, cx, cy, col, alpha) pcall(function() d.PointA = vector(ax, ay) d.PointB = vector(bx, by)
d.PointC = vector(cx, cy) d.Color = col d.Transparency = alpha d.Visible = true end) V.pc(d).vis = true end V.circ = function(d, x, y, r, col, alpha) pcall(function() d.Position = vector(x, y) d.Radius = r d.Color = col
d.Transparency = alpha d.Visible = true end) V.pc(d).vis = true end V.draw_dot_cursor = function(mx, my, col, hot) V.hide(V.cur_glow); V.hide(V.cur_rip); V.hide(V.cur_rip2); V.hide(V.cur_shine)
for i = 1, 6 do V.hide(V.cur_trail[i]) end for i = 1, V.CUR_SL do V.hide(V.cur_so[i]); V.hide(V.cur_sf[i]) end V.circ(V.cur_rip, mx, my, 3.2, color(6, 8, 14), 0.9) pcall(function() V.cur_rip.Filled = true end)
V.circ(V.cur_dot, mx, my, 2, hot and V.C_WHITE or col, 1) end V.draw_cursor = function(mx, my, A, A2, pressed, hot, now, dt) local C = V.CUR C.hot = V.m_lerp(C.hot, hot and 1 or 0, math.min(1, dt * 14)) if pressed then
C.press_t = now C.rip_t = now C.rip_x = mx C.rip_y = my end C.tn = (C.tn % 6) + 1 C.trail[C.tn] = { mx, my } for i = 1, 6 do local idx = (((C.tn - i) % 6) + 1) local pt = C.trail[idx] local tdot = V.cur_trail[i]
if pt and i > 1 then local f = 1 - (i / 7) V.circ(tdot, pt[1] + 1, pt[2] + 1, 1 + (2.2 * f), A, 0.35 * f) else V.hide(tdot) end end local pk = 1 - V.ease((now - C.press_t) / 0.18) local sc = 1 - (0.14 * pk)
local L = 19 * sc local W = 13 * sc local fill = V.mix(A, A2, C.hot) local sh = L / V.CUR_SL for i = 1, V.CUR_SL do local y0 = (i - 1) * sh local wmid = V.arrow_w(y0 + (sh / 2), L, W) local wf = math.max(1, wmid)
V.rect(V.cur_so[i], mx - 1.5, (my + y0) - 1.5, wf + 3, sh + 3, color(6, 8, 14)) V.rect(V.cur_sf[i], mx, my + y0, wf, sh + 0.5, V.mix(fill, A2, (i / V.CUR_SL) * 0.35)) end
V.rect(V.cur_shine, mx + 1, my + 3, 2, L * 0.5, V.C_WHITE) V.alpha(V.cur_shine, 0.55) C.spin = C.spin + (dt * 3) local orb = 7 + (2 * C.hot)
V.circ(V.cur_dot, ((mx + W) + 2) + (math.cos(C.spin) * 2), ((my + L) - 1) + (math.sin(C.spin) * 2), 1.6 + C.hot, A2, 0.6 + (0.4 * C.hot)) if C.hot > 0.03 then V.circ(V.cur_glow, mx + 4, my + 6, orb + 5, A, 0.14 * C.hot)
else V.hide(V.cur_glow) end local ra = (now - C.rip_t) / 0.4 if ra < 1 then local e = V.ease(ra) V.circ(V.cur_rip, C.rip_x, C.rip_y, 3 + (16 * e), A2, 0.85 * (1 - e))
V.circ(V.cur_rip2, C.rip_x, C.rip_y, 2 + (9 * e), V.C_WHITE, 0.5 * (1 - e)) else V.hide(V.cur_rip); V.hide(V.cur_rip2) end end V.m_x = math.floor((V.sw - V.MW) / 2) V.m_y = math.floor((V.sh - V.MH) / 2) V.m_tab = _v(1)
V.m_tab_t = _v(0) V.m_tanim = {} V.m_thov = {} for i = 1, V.UI_TN do V.m_tanim[i] = (i == 1) and 1 or 0; V.m_thov[i] = 0 end V.m_drag = _v(false) V.m_dx = _v(0) V.m_dy = _v(0) V.m_slider = _v(nil) V.m_dd = _v(nil)
V.m_dd_t = _v(0) V.m_prev = _v(false) V.m_hot = _v(false) V.m_hide_slot = function(s) V.hide(s.bg); V.hide(s.lb); V.hide(s.a); V.hide(s.b); V.hide(s.c); V.hide(s.p); V.hide(s.v); V.hide(s.bx); V.hide(s.bt); V.hide(s.ar)
end V.M_STATIC = { V.m_shadow, V.m_win, V.m_side, V.m_sidefix, V.m_sideln, V.m_logo_glow, V.m_logo, V.m_logo_in, V.m_logo_t, V.m_logo_dot, V.m_title_a, V.m_title, V.m_sub,
V.m_chip1, V.m_chip1_i, V.m_chip1_t, V.m_chip1_a, V.m_chip2, V.m_chip2_t, V.m_gh[1], V.m_gh[2], V.m_card[1], V.m_card[2],
V.m_ubg, V.m_uav, V.m_uav_t, V.m_un, V.m_us, V.m_uch, V.m_ddshadow, V.m_ddbg, V.m_ddhl, V.m_cursor } V.m_hide_all = function() for _, d in ipairs(V.M_STATIC) do V.hide(d) end for r = 1, V.SIDE_N do
V.hide(V.m_sh[r]); V.hide(V.m_sbg[r]); V.hide(V.m_stx[r]); V.hide(V.m_sarr[r]) V.icon_hide(V.m_sic[r]) end for i = 1, V.M_SLOTS do V.m_hide_slot(V.m_slot[i]); V.m_slot[i].kind = nil end
for i = 1, V.M_DD do V.hide(V.m_ddtx[i]) end for _, d in ipairs(V.M_EXTRA) do V.hide(d) end V.PF2.k = 0 V.TIP.k = 0 V.SR.focus = false V.OV_SR = false V.OV_PF = false V.cur_hide() end V.ICONS = {
["Anti-Aim"] = { { 0, 3, 9, 2, 1 }, { 8, 1, 2, 6, 1 }, { 10, 2, 2, 4, 1 }, { 5, 9, 9, 2, 1 }, { 4, 7, 2, 6, 1 }, { 2, 8, 2, 4, 1 } }, ["AA States"] = { { 1, 0, 12, 3, 1 }, { 0, 5, 14, 4, 2 }, { 1, 11, 12, 3, 1 } },
["Defensive"] = { { 1, 0, 12, 7, 2 }, { 2, 6, 10, 4, 3 }, { 4, 9, 6, 3, 2 }, { 6, 11, 2, 3, 1 } }, ["Hard Flick"] = { { 8, 0, 5, 4, 1 }, { 6, 3, 5, 3, 1 }, { 3, 6, 9, 2, 1 }, { 4, 8, 5, 3, 1 }, { 2, 11, 4, 3, 1 } },
["Resolver"] = { { 6, 0, 2, 4, 1 }, { 6, 10, 2, 4, 1 }, { 0, 6, 4, 2, 1 }, { 10, 6, 4, 2, 1 }, { 5, 5, 4, 4, 2 } },
["Rage"] = { { 0, 6, 4, 8, 0 }, { 0, 2, 4, 7, 2 }, { 5, 5, 4, 9, 0 }, { 5, 0, 4, 8, 2 }, { 10, 6, 4, 8, 0 }, { 10, 2, 4, 7, 2 } }, ["Visuals"] = { { 0, 0, 14, 9, 2 }, { 6, 9, 2, 3, 0 }, { 3, 12, 8, 2, 1 } },
["Players"] = { { 4, 0, 6, 6, 3 }, { 1, 7, 12, 7, 3 } }, ["Extras"] = { { 6, 0, 2, 14, 1 }, { 0, 6, 14, 2, 1 }, { 4, 4, 6, 6, 2 }, { 11, 0, 3, 3, 1 } },
["Music"] = { { 9, 0, 2, 11, 1 }, { 2, 8, 9, 6, 3 }, { 9, 0, 5, 3, 1 }, { 12, 2, 2, 3, 1 } }, ["Grenades"] = { { 2, 4, 10, 10, 5 }, { 4, 2, 6, 3, 1 }, { 8, 0, 4, 3, 1 }, { 11, 2, 2, 5, 1 } },
["Binds"] = { { 0, 2, 4, 4, 1 }, { 5, 2, 4, 4, 1 }, { 10, 2, 4, 4, 1 }, { 2, 8, 10, 4, 1 } }, ["Config"] = { { 0, 2, 14, 2, 1 }, { 2, 0, 5, 6, 2 }, { 0, 10, 14, 2, 1 }, { 8, 8, 5, 6, 2 } }, }
V.set_prop = function(d, k, v) d[k] = v end V.icon_part = function(d, pt, x, y, col) if not pt then V.hide(d) return end V.rect(d, x + pt[1], y + pt[2], pt[3], pt[4], col) local c = V.PC[d] or V.pc(d)
local rnd = math.max(0, math.min(pt[5], math.floor(math.min(pt[3], pt[4]) / 2))) if c.rnd ~= rnd then c.rnd = rnd pcall(V.set_prop, d, "Rounding", rnd) end if not c.fill then c.fill = true
pcall(V.set_prop, d, "Filled", true) end end V.draw_icon = function(parts, name, x, y, col) local fx = math.floor(x) local fy = math.floor(y)
if parts.dn == name and parts.dx == fx and parts.dy == fy and parts.dc == col and parts[1] and V.PC[parts[1]] and V.PC[parts[1]].vis then return end parts.dn = name parts.dx = fx parts.dy = fy parts.dc = col
local def = V.ICONS[name] or {} V.icon_part(parts[1], def[1], x, y, col) V.icon_part(parts[2], def[2], x, y, col) V.icon_part(parts[3], def[3], x, y, col) V.icon_part(parts[4], def[4], x, y, col)
V.icon_part(parts[5], def[5], x, y, col) V.icon_part(parts[6], def[6], x, y, col) end V.m_in = function(mx, my, x, y, w, h) return mx >= x and mx <= (x + w) and my >= y and my <= (y + h) end V.m_lerp = function(a, b, k)
local d = b - a if d < 0.002 and d > -0.002 then return b end return a + (d * k) end V.ui_set_open = function(o) V.ui_open = o V.ui_open_t = V.now_s() V.m_drag = false V.m_slider = nil V.m_dd = nil
if o then V.ui_hint_done = true end if o and not V.reveal_t then V.reveal_t = V.now_s() + 0.12 end pcall(input.set_cursor_unlocked, o) end V.TIPS = {
["+ Per Spread Miss"] = { "Hitchance added after each spread miss on the same target", "Сколько хитшанса добавить после каждого промаха по спреду" },
["AA Classifier"] = { "Detects each enemy's AA type: static, jitter, spin, defensive, no aa", "Определяет тип AA врага: static, jitter, spin, defensive, no aa" },
["AA Preset"] = { "Ready-made AA setup to apply with the button below", "Готовая настройка AA, применяется кнопкой ниже" },
["AA While Shooting"] = { "Disable: drops AA while the aimbot has a target, so it shoots freely", "Disable: снимает AA, пока у аимбота есть цель, чтобы он стрелял смело" },
["AA-Off Detection"] = { "Enemies with ~0 desync are marked no aa and shot without brute", "Врагов без десинка помечает no aa и стреляет без перебора" },
["Accent Color"] = { "Main color of menu, panels and indicators", "Основной цвет меню, панелей и индикаторов" },
["Accurate Shooting"] = { "Stops flicks and fake lag around your shots", "Выключает флики и фейклаг вокруг твоих выстрелов" },
["Adaptive Hitchance"] = { "Raises hitchance after spread misses (overrides cheat value)", "Поднимает хитшанс после промахов по спреду (перекрывает чит)" },
["Adaptive Lag"] = { "Max fake lag while moving or in air", "Максимальный фейклаг в движении и в прыжке" },
["Angle A"] = { "First desync limit Hard Flick switches between", "Первый лимит десинка, между которыми скачет Hard Flick" },
["Angle B"] = { "Second desync limit Hard Flick switches between", "Второй лимит десинка, между которыми скачет Hard Flick" },
["Angle Memory"] = { "Brute starts from angles that already hit this enemy", "Перебор начинается с углов, по которым уже попадал" },
["Announce Headshots"] = { "Banner on every headshot kill", "Баннер на каждый килл в голову" }, ["Announcer Sound"] = { "Sound played with the killstreak banner", "Звук вместе с баннером стрика" },
["Anti-Aim Toggle"] = { "Key that turns Lua AA on / off", "Клавиша вкл / выкл AA луа" },
["Anti-Brute Mode"] = { "Random: random angles after a hit on you. Stages: step through limits", "Random: случайные углы после попадания по тебе. Stages: по ступеням" },
["Anti-Bruteforce"] = { "Changes your AA when an enemy hits you", "Меняет твой AA, когда по тебе попали" }, ["Apply Preset"] = { "Applies the selected AA preset", "Применяет выбранный AA-пресет" },
["BAim After Misses"] = { "Prefer / force body after this many resolver misses on a target", "Бить в тело после стольких промахов резольвера по цели" },
["BAim Mode"] = { "Prefer: body first, head still allowed. Force: body only", "Prefer: сначала тело, голова тоже можно. Force: только тело" },
["Base Hitchance"] = { "Starting hitchance for Adaptive Hitchance", "Стартовый хитшанс для Adaptive Hitchance" }, ["Bee Count"] = { "How many bees fly around you", "Сколько пчёл летает вокруг" },
["Bee Orbit"] = { "Orbit radius in studs", "Радиус орбиты в стадах" }, ["Bee Size"] = { "Bee size on screen", "Размер пчёл на экране" }, ["Bee Style"] = { "Emoji or drawn bees", "Эмодзи или нарисованные пчёлы" },
["Bees"] = { "Bees orbiting your character", "Пчёлы, которые кружат вокруг тебя" }, ["Bees Swarm Kills"] = { "The swarm flies to every enemy you kill", "Рой летит к каждому убитому врагу" },
["Body First vs Defensive"] = { "Body priority vs enemies using defensive flicks", "Приоритет тела против врагов с defensive-фликами" },
["Break LC in Air"] = { "Shifts a tick in air to break lag compensation", "Сдвиг тика в прыжке, ломает лагкомп" },
["Built-in Mode Switch"] = { "Cycles the cheat's built-in resolver mode", "Переключает режим встроенного резольвера чита" },
["Config Slot"] = { "Slot for Save / Load (cleared on lua reload, use codes to keep)", "Слот для Save / Load (чистится при релоаде, храни кодом)" },
["Crosshair Indicators"] = { "AA state text under the crosshair", "Текст состояния AA под прицелом" }, ["Cull Crosshair In Scope"] = { "Hide crosshair indicators while scoped", "Прятать индикаторы у прицела в скопе" },
["Cull Max 3 Hit Logs"] = { "Show at most 3 hit logs", "Не больше 3 хит-логов на экране" }, ["DT Bar"] = { "Double tap charge bar, drag it with the menu open", "Полоса заряда DT, двигается мышкой при открытом меню" },
["DT Bar X"] = { "DT bar horizontal position", "Позиция DT-полосы по горизонтали" }, ["DT Bar Y"] = { "DT bar vertical position", "Позиция DT-полосы по вертикали" },
["Damage-Validated Hits"] = { "Weak head hits count as half-misses (you hit a fake)", "Слабые хиты в голову считаются полупромахом (попал в фейк)" },
["Defensive Flicks"] = { "Short pitch / yaw flicks to break enemy aim", "Короткие флики pitch / yaw, ломают прицел врагу" },
["Disable Built-in"] = { "Turns off the cheat resolver while Lua Brute works", "Выключает резольвер чита, пока работает Lua Brute" },
["Drag Watermark"] = { "Move the watermark with the mouse", "Двигать ватермарку мышкой" },
["Dynamic Min Damage"] = { "Lowers min damage only when the enemy can be finished", "Снижает MinDmg только когда врага можно добить" },
["Edit State"] = { "Which movement state you are editing", "Какое состояние движения редактируешь" }, ["Enable AA"] = { "Lua anti-aim on / off", "Вкл / выкл AA луа" },
["Enable AA States"] = { "Separate AA for standing, moving, air, crouch", "Отдельный AA для стойки, бега, прыжка, приседа" },
["Enable Fake Lag"] = { "Fake lag. Off = cheat fake lag forced off too", "Фейклаг. Off = фейклаг чита тоже выключен" },
["Enable Hard Flick"] = { "Rapidly switches desync between Angle A and B", "Быстро переключает десинк между Angle A и B" },
["Enable Lua Resolver"] = { "Resolver assist on top of the cheat", "Помощь резольверу поверх чита" }, ["Enemy Labels"] = { "Label above enemies with AA type and misses", "Метка над врагами: тип AA и промахи" },
["Enemy Pitch"] = { "Override enemy pitch for the aimbot", "Подмена pitch врага для аимбота" }, ["Flick Interval"] = { "Ticks between defensive flicks", "Тиков между defensive-фликами" },
["Flick Length"] = { "How long a flick lasts in ticks", "Сколько тиков длится флик" }, ["Flick Speed"] = { "How often Hard Flick switches", "Как часто переключается Hard Flick" },
["Flick Trigger"] = { "Timer, on peek, or both", "По таймеру, на пике или оба" }, ["Flick on Own Shot"] = { "Flick right after you shoot", "Флик сразу после твоего выстрела" },
["Font"] = { "Font for all lua text", "Шрифт всего текста луа" }, ["Force Safe Point"] = { "Always shoot safe points", "Всегда стрелять в safe point" }, ["HC Air"] = { "Hitchance while in air", "Хитшанс в прыжке" },
["HC Run"] = { "Hitchance while running", "Хитшанс при беге" }, ["HC Walk"] = { "Hitchance while walking", "Хитшанс при ходьбе" },
["Hard After Misses"] = { "Mark a target hard after this many total misses", "Пометить цель HARD после стольких промахов" },
["Highlight Target"] = { "Highlight the aimbot target in the list", "Подсветка цели аимбота в списке" }, ["Hit Logs"] = { "Hit / miss cards under the crosshair", "Карточки хитов и промахов под прицелом" },
["Hitbox Priority"] = { "Menu: use cheat. Body / Head First overrides it", "Menu: как в чите. Body / Head First перекрывают" }, ["Intro Animation"] = { "NeverPeek intro on load", "Интро NeverPeek при загрузке" },
["Intro Dim + Vignette"] = { "Darken screen during the intro", "Затемнение экрана во время интро" },
["Inverter"] = { "How desync side switches: jitter, random, static", "Как меняется сторона десинка: jitter, random, static" },
["Jitter Detection"] = { "Old jitter detector (the classifier does it better)", "Старый детектор jitter (классификатор лучше)" },
["Keybinds List"] = { "Panel with binds and live states", "Панель биндов и их состояний" }, ["Kill Say"] = { "Writes a line in chat after kills", "Пишет фразу в чат после киллов" },
["Kill Say Cooldown"] = { "Minimum seconds between messages", "Минимум секунд между сообщениями" }, ["Kill Say Language"] = { "Kill say language", "Язык фраз" },
["Kill Say On"] = { "Which kills trigger kill say", "На какие киллы писать" }, ["Killstreak Announcer"] = { "Big banner on multi-kills", "Большой баннер на мультикиллы" },
["Lag Limit"] = { "Max choked ticks", "Сколько тиков максимум копить" }, ["Lag Variability"] = { "Random spread of fake lag", "Случайный разброс фейклага" },
["Lag While Shooting"] = { "Fake lag ticks during shots", "Фейклаг во время выстрелов" }, ["Left Limit"] = { "Max desync to the left", "Максимальный десинк влево" },
["Lethal BAim"] = { "Body when one body shot kills", "Тело, когда один выстрел в тело добивает" }, ["Limit Jitter"] = { "Swap or randomize left / right limits", "Менять местами или рандомить лимиты" },
["Limit Jitter Speed"] = { "Ticks between limit changes", "Тиков между сменами лимитов" }, ["List X"] = { "Players list horizontal position", "Позиция списка игроков по горизонтали" },
["List Y"] = { "Players list vertical position", "Позиция списка игроков по вертикали" },
["Load Code"] = { "Load config from the NeverPeek code box in the cheat menu", "Загрузить конфиг из поля NeverPeek code в меню чита" }, ["Load Config"] = { "Load the selected slot", "Загрузить выбранный слот" },
["Log Shots To Console"] = { "HIT / MISS lines with reasons in console", "Строки HIT / MISS с причинами в консоли" }, ["Manual Inverter"] = { "Key that flips desync side", "Клавиша смены стороны десинка" },
["Max Enemy Desync"] = { "Biggest angle the brute tries", "Самый большой угол перебора" }, ["Max Hitchance"] = { "Cap for Adaptive Hitchance", "Потолок для Adaptive Hitchance" },
["Menu Key"] = { "Key that opens this menu", "Клавиша открытия меню" }, ["Miss Breakdown"] = { "Miss reasons per player in the list", "Причины промахов по игрокам в списке" },
["Modifier Hold"] = { "Ticks each yaw angle is held", "Сколько тиков держать каждый угол yaw" }, ["Modifier Range"] = { "Yaw jitter width in degrees", "Ширина джиттера yaw в градусах" },
["Movement Hitchance"] = { "Own hitchance while moving (ssg / deagle)", "Свой хитшанс в движении (ssg / deagle)" },
["Multi-kill Window"] = { "Seconds between kills to count a multi-kill", "Секунд между киллами для мультикилла" },
["Override Global"] = { "Use this state's settings instead of main AA", "Брать настройки этого состояния вместо общих" },
["Peek Flick Window"] = { "Flick ticks after a peek is detected", "Тиков фликов после обнаружения пика" }, ["Per-Type Strategy"] = { "Different brute and baim per AA type", "Свой перебор и baim под каждый тип AA" },
["Perf Profiler"] = { "Prints ms per part of the lua to console every 5 s", "Раз в 5 с пишет в консоль мс по частям луа" },
["Performance"] = { "Quality / Balanced / Potato redraw rate", "Частота перерисовки: Quality / Balanced / Potato" }, ["Pitch Up / Down"] = { "Key that flips pitch", "Клавиша смены pitch" },
["Player State"] = { "Your movement state under the crosshair", "Твоё состояние движения под прицелом" }, ["Players List"] = { "Panel with every enemy: AA, angle, hits, HP", "Панель врагов: AA, угол, хиты, HP" },
["Pollen Trail"] = { "Little yellow dots behind bees", "Жёлтые точки за пчёлами" }, ["Position X"] = { "Watermark horizontal position", "Позиция ватермарки по горизонтали" },
["Position Y"] = { "Watermark vertical position", "Позиция ватермарки по вертикали" },
["Save Code"] = { "Saves your config as a code into the cheat menu box (survives reload)", "Сохраняет конфиг кодом в поле меню чита (переживает релоад)" },
["Random A (0-86)"] = { "Random angle A each switch", "Случайный угол A при каждом переключении" }, ["Random B (0-86)"] = { "Random angle B each switch", "Случайный угол B при каждом переключении" },
["Randomize Hold"] = { "Random hold time up to Modifier Hold", "Случайная задержка до Modifier Hold" },
["Remember Hard Targets"] = { "Remember enemies that keep dodging", "Запоминать врагов, по которым всё время мимо" },
["Remember Hit Angle"] = { "Keep the angle that hit until it misses", "Держать угол, который попал, пока не промажет" }, ["Reset Stats"] = { "Reset hit rate and miss stats", "Сбросить хитрейт и статистику" },
["Reset To Default"] = { "Reset every setting to default", "Сбросить все настройки" }, ["Resolver Mode"] = { "Assist the cheat resolver or full Lua brute", "Помогать резольверу чита или полный перебор луа" },
["Right Limit"] = { "Max desync to the right", "Максимальный десинк вправо" }, ["Safe Point After"] = { "Force safe point after this many misses", "Safe point после стольких промахов" },
["Save Config"] = { "Save to the selected slot", "Сохранить в выбранный слот" }, ["Scope Delay"] = { "Extra wait after scoping in", "Доп. ожидание после скопа" },
["Show Dead Players"] = { "Keep dead enemies in the list", "Показывать мёртвых в списке" }, ["Smart DT"] = { "Never burns DT charge when a fight can start", "Не тратит заряд DT, когда может начаться перестрелка" },
["Speed Mode"] = { "Fixed speed or wave", "Фиксированная скорость или волна" }, ["Spin Duration"] = { "How long pitch spins after a shot", "Сколько крутится pitch после выстрела" },
["Spin Pitch On Shot"] = { "Spin pitch right after shooting", "Крутить pitch после выстрела" }, ["Stage Reset"] = { "Seconds until anti-brute stages reset", "Через сколько секунд сбросить ступени" },
["Stats Panel"] = { "Hit rate panel under the watermark", "Панель хитрейта под ватермаркой" }, ["Test Chat"] = { "Sends a test kill say line", "Отправить тестовую фразу" },
["Tickbase Shift"] = { "Shift ticks on flicks (eats DT, Smart DT guards it)", "Сдвиг тиков на фликах (ест DT, Smart DT бережёт)" }, ["Tips Language"] = { "Language of these hints", "Язык этих подсказок" },
["Wait For Scope"] = { "Don't shoot until fully scoped", "Не стрелять, пока не открылся скоп" }, ["Watermark"] = { "Top-left watermark with fps / ping", "Ватермарка с fps / ping" },
["Wave Cycle"] = { "Wave cycles per second", "Циклов волны в секунду" }, ["Wave Fast Speed"] = { "Fastest speed in wave mode", "Самая быстрая скорость в волне" },
["Yaw Modifier"] = { "How your yaw moves: mixed, jitter, 3-way, random, sway", "Как двигается yaw: mixed, jitter, 3-way, random, sway" }, ["Yaw Offset"] = { "Static yaw offset in degrees", "Постоянное смещение yaw" }, }
V.TIPS["Grenade Prediction"] = { "Shows where your HE / molotov lands while you hold it", "Показывает, куда упадёт HE / молик, пока держишь его" }
V.TIPS["Throw Strength"] = { "Auto: LMB full, RMB short, both medium", "Auto: ЛКМ полный, ПКМ короткий, обе средний" }
V.TIPS["Velocity Inheritance"] = { "Adds your movement speed to the throw", "Учитывает твою скорость в броске" }
V.TIPS["Draw Through Walls"] = { "Arcs and zones visible through walls", "Траектории и зоны видно сквозь стены" }
V.TIPS["Zones Only On Ground"] = { "Hide damage zones for nades that end in the air", "Не рисовать зону, если граната взорвётся в воздухе" }
V.TIPS["Damage Labels"] = { "Expected damage / KILL above enemies in the zone", "Ожидаемый урон / KILL над врагами в зоне" }
V.TIPS["Track Thrown Grenades"] = { "Arc, fuse timer and zone for every flying nade", "Траектория, таймер и зона для каждой летящей гранаты" }
V.TIPS["Molotov Fire Zones"] = { "Fire area on the ground with time left", "Зона огня на земле и сколько осталось гореть" } V.TIPS["Fire Lifetime"] = { "How long molotov fire burns", "Сколько горит молик" }
V.TIPS["Fire Radius"] = { "Molotov fire radius in studs", "Радиус огня молика в стадах" } V.TIPS["HE Radius"] = { "HE damage radius in studs", "Радиус урона HE в стадах" }
V.TIPS["HE Damage"] = { "HE damage at the center", "Урон HE в центре" }
V.TIPS["Bullet Tracers"] = { "Line from you to where each bullet stopped, dots = wall hits", "Линия от тебя до места, где остановилась пуля, точки = стены" }
V.TIPS["Tracer Time"] = { "How long tracers stay on screen", "Сколько держатся трейсеры" }
V.TIPS["Anim Resolver"] = { "Uses enemy animations (feet vs eye yaw) as the first angle to shoot", "Берёт угол из анимаций врага (ноги против взгляда) как первый угол" }
V.TIPS["Intro Music"] = { "Plays a track while loading, fades out when you open the menu", "Играет трек при загрузке, затухает при открытии меню" } V.TIPS["Music Volume"] = { "Intro music volume", "Громкость музыки" }
V.TIPS["Show Advanced"] = { "Shows niche settings and the Hard Flick tab", "Показывает редкие настройки и вкладку Hard Flick" } V.TAB_TIPS = { ["Anti-Aim"] = { "Main anti-aim settings", "Основные настройки AA" },
["AA States"] = { "Separate AA per movement state", "AA по состояниям движения" }, ["Defensive"] = { "Flicks, tickbase and shooting behaviour", "Флики, тикбейз и поведение при стрельбе" },
["Hard Flick"] = { "Rapid desync switching", "Быстрое переключение десинка" }, ["Resolver"] = { "Resolver assist and AA classifier", "Помощь резольверу и классификатор AA" },
["Rage"] = { "Hitchance, min damage, baim, DT, fake lag", "Хитшанс, MinDmg, baim, DT, фейклаг" }, ["Visuals"] = { "Panels, fonts, colors, performance", "Панели, шрифты, цвета, производительность" },
["Players"] = { "Enemy list settings", "Настройки списка врагов" }, ["Extras"] = { "Announcer, kill say, labels, bees", "Анонсер, kill say, метки, пчёлы" },
["Grenades"] = { "Grenade prediction, live nades, molotov fire zones", "Предикт гранат, гранаты в полёте, зоны молика" }, ["Binds"] = { "Keys", "Клавиши" },
["Config"] = { "Save, load, share configs and presets", "Сохранение, загрузка, коды и пресеты" }, } V.M_EXTRA = {} V.m_act_t = -10 V.m_full_t = -10 V.m_hot_prev = false V.m_lmx = -1 V.m_lmy = -1
V.MX = function(d) V.M_EXTRA[#V.M_EXTRA + 1] = d return d end V.tip_bg = V.MX(V.MK(8, 3)) V.tip_ic = V.MX(V.MK(8, 4)) V.tip_q = V.MX(V.MT(12, 5, true, true)) V.tip_tx = V.MX(V.MT(12, 5))
V.TIP = { k = 0, w = false, txt = "", t = -10 } V.tip_text = function() local L = (V.tips_lang:get() == "RU") and 2 or 1 local w = V.m_hover_w if w and w.name then local key = w.name local e = V.TIPS[key]
if not e and w.key then e = V.TIPS[w.key] end if e then return e[L] end return false end if V.m_hover_tab then local tn = V.UI_TABS[V.m_hover_tab] local e = tn and V.TAB_TIPS[tn.name] if e then return e[L] end end
return false end V.draw_tip = function(x, y, k, A, C, ks) local txt = V.tip_text() if txt then V.TIP.txt = txt end V.TIP.k = V.m_lerp(V.TIP.k, (txt and k > 0.5) and 1 or 0, ks) local a = V.TIP.k * k if a < 0.03 then
if not V.TIP.hidden then V.TIP.hidden = true V.hide(V.tip_bg); V.hide(V.tip_ic); V.hide(V.tip_q); V.hide(V.tip_tx) end return end V.TIP.hidden = false local cx0 = (x + V.MSIDE) + V.MPAD
local ty = ((y + V.MH) - 40) + ((1 - V.TIP.k) * 6) V.rect(V.tip_bg, cx0, ty, V.MCW, 28, C.card, C.cardb) V.alpha(V.tip_bg, a) V.rect(V.tip_ic, cx0 + 8, ty + 6, 16, 16, A) V.alpha(V.tip_ic, a)
V.text(V.tip_q, cx0 + 16, ty + 7, "?", color(10, 12, 18)) V.alpha(V.tip_q, a) V.text(V.tip_tx, cx0 + 32, ty + 7, V.TIP.txt, C.text) V.alpha(V.tip_tx, a) end
V.SR = { focus = false, q = "", res = {}, rn = 0, last_q = false, hl = 0 } V.SR_W = _v(196) V.sr_box = V.MX(V.MK(6, 3)) V.sr_ic1 = V.MX(V.MK(4, 4)) V.sr_ic2 = V.MX(V.MK(1, 4)) V.sr_tx = V.MX(V.MT(12, 5))
V.sr_car = V.MX(V.MK(0, 5)) V.sr_sh = V.MX(V.MK(10, 9)) V.sr_bg = V.MX(V.MK(8, 10)) V.sr_hl = V.MX(V.MK(6, 11)) V.SR_N = _v(8) V.sr_rt = {} V.sr_rs = {} for i = 1, V.SR_N do V.sr_rt[i] = V.MX(V.MT(12, 12))
V.sr_rs[i] = V.MX(V.MT(11, 12)) end V.sr_find = function(q) local out = {} if q == "" then return out end for i = 1, V.UI_TN do local sec = V.UI_TABS[i] for j = 1, sec.n do local w = sec.items[j]
if #out < V.SR_N and not w.hidden and w.t ~= "label" and string.find(string.lower(w.name), q, 1, true) then out[#out + 1] = { tab = i, w = w } end end end return out end V.sr_key = function(k) local S = V.SR
if k == "Escape" or k == "Return" then S.focus = false return true end if k == "Backspace" then S.q = string.sub(S.q, 1, math.max(0, string.len(S.q) - 1)) return true end
if k == "Space" then if string.len(S.q) < 20 then S.q = S.q .. " " end return true end if string.len(k) == 1 and string.match(k, "%a") then if string.len(S.q) < 20 then S.q = S.q .. string.lower(k) end return true end
local DIG = { Zero = "0", One = "1", Two = "2", Three = "3", Four = "4", Five = "5", Six = "6", Seven = "7", Eight = "8", Nine = "9", Minus = "-" } if DIG[k] then if string.len(S.q) < 20 then S.q = S.q .. DIG[k] end
return true end return true end V.sr_hide_res = function() if V.SR.res_hidden then return end V.SR.res_hidden = true V.hide(V.sr_sh); V.hide(V.sr_bg); V.hide(V.sr_hl)
for i = 1, V.SR_N do V.hide(V.sr_rt[i]); V.hide(V.sr_rs[i]) end end V.draw_search = function(x, y, mx, my, pressed, now, A, C) local S = V.SR local sx = ((x + V.MSIDE) + V.MPAD) + 144 local sy = y + 12 local W = V.SR_W
local hov = V.m_in(mx, my, sx, sy, W, 30) if hov then V.m_hot = true end if V.ui_open and pressed then if hov then S.focus = true
elseif S.focus and not V.m_in(mx, my, sx, sy + 34, 300, (V.SR_N * 26) + 10) then S.focus = false end end if not V.ui_open then S.focus = false end
V.rect(V.sr_box, sx, sy, W, 30, (hov or S.focus) and C.sel or C.chip, S.focus and A or C.cardb) V.rect(V.sr_ic1, sx + 11, sy + 9, 9, 9, S.focus and A or C.dim)
V.rect(V.sr_ic2, sx + 18, sy + 17, 5, 2, S.focus and A or C.dim) local shown = S.q if shown == "" and not S.focus then shown = "search settings" end local tc = (S.q == "") and C.dim or C.text
V.text(V.sr_tx, sx + 30, sy + 8, shown, tc) if S.focus and (math.floor(now * 2) % 2) == 0 then local cw = (S.q == "") and 0 or V.tw(S.q, 12) V.rect(V.sr_car, (sx + 31) + cw, sy + 8, 1, 14, A) else V.hide(V.sr_car) end
if S.q ~= S.last_q then S.last_q = S.q S.res = V.sr_find(S.q) S.rn = #S.res end if not (S.focus and S.rn > 0) then V.sr_hide_res() V.OV_SR = false return end S.res_hidden = false local rx = sx local ry = sy + 36
local rw = 300 local rh = (S.rn * 26) + 8 V.OV_SR = { rx, ry, rw, rh } V.rect(V.sr_sh, rx - 3, ry - 1, rw + 6, rh + 6, C.shadow) V.alpha(V.sr_sh, 0.35) V.rect(V.sr_bg, rx, ry, rw, rh, C.chip, C.cardb) local any = false
for i = 1, V.SR_N do local r = S.res[i] if r then local iy = (ry + 4) + ((i - 1) * 26) local h = V.m_in(mx, my, rx, iy, rw, 26) if h then any = true V.rect(V.sr_hl, rx + 4, iy, rw - 8, 26, V.mix(C.chip, A, 0.25))
if pressed then V.m_tab = r.tab V.m_tab_t = now V.m_slider = nil r.w.flash_t = now S.focus = false S.q = "" end end V.text(V.sr_rt[i], rx + 12, iy + 5, r.w.name, h and V.C_WHITE or C.text)
local tn = V.UI_TABS[r.tab].name V.text(V.sr_rs[i], ((rx + rw) - 12) - V.tw(tn, 11), iy + 6, tn, C.dim) else V.hide(V.sr_rt[i]); V.hide(V.sr_rs[i]) end end if not any then V.hide(V.sr_hl) end end
V.PF2 = { open = false, k = 0, t0 = false } V.SESSION_T = V.now_s() V.pf_o = {} V.pf_obj = function(name, d) V.pf_o[name] = V.MX(d) return d end
V.pf_obj("sh", V.MK(14, 13)); V.pf_obj("bg", V.MK(12, 14)); V.pf_obj("ln", V.MK(1, 15)) V.pf_obj("ring", V.MK(10, 15)); V.pf_obj("av", V.MK(9, 16)); V.pf_obj("avt", V.MT(20, 17, true, true))
V.pf_obj("nm", V.MT(15, 16, false, true)); V.pf_obj("un", V.MT(12, 16)); V.pf_obj("id", V.MT(11, 16)) V.pf_obj("rk", V.MK(6, 15)); V.pf_obj("rkt", V.MT(11, 16, true, true))
V.pf_obj("bb", V.MK(2, 15)); V.pf_obj("bf", V.MK(2, 16)); V.pf_obj("nx", V.MT(10, 16)) V.pf_obj("cell", V.MK(8, 15)); V.pf_obj("ft", V.MT(10, 16)) V.pf_sl = {} V.pf_sv = {} for i = 1, 6 do V.pf_sl[i] = V.MX(V.MT(10, 16))
V.pf_sv[i] = V.MX(V.MT(15, 16, false, true)) end V.pf_sep1 = V.MX(V.MK(0, 16)) V.pf_sep2 = V.MX(V.MK(0, 16)) V.RANKS = { { 0, "ROOKIE", color(150, 160, 185) }, { 5, "PEEKER", color(90, 200, 255) },
{ 15, "NEVER PEEKER", color(150, 110, 255) }, { 30, "HEADHUNTER", color(255, 90, 110) }, { 50, "GODLIKE", color(255, 196, 60) }, } V.rank_of = function(k) local idx = 1 for i, r in ipairs(V.RANKS) do
if k >= r[1] then idx = i end end return idx end V.pf_hide = function() if V.PF2.hidden then return end V.PF2.hidden = true for _, d in pairs(V.pf_o) do V.hide(d) end
for i = 1, 6 do V.hide(V.pf_sl[i]); V.hide(V.pf_sv[i]) end V.hide(V.pf_sep1); V.hide(V.pf_sep2) end V.lp_info = function() local lp = V.find_lp()
return { name = tostring(lp.Name), dn = tostring(lp.DisplayName), id = tonumber(lp.UserId) or 0, age = tonumber(lp.AccountAge) or 0 } end V.draw_profile = function(x, y, uy, mk, now, A, C, ks) local P = V.PF2
P.k = V.m_lerp(P.k, (P.open and V.ui_open) and 1 or 0, ks) local k = P.k * mk if k < 0.03 then V.pf_hide() V.OV_PF = false return end V.PF2.hidden = false local o = V.pf_o local W = 300 local H = 236
local px = ((x + V.MSIDE) + 10) - ((1 - V.ease(P.k)) * 24) local py = (uy + 50) - H V.OV_PF = { px, py, W, H } if not P.info or (now - (P.info_t or -10)) > 5 then local ok, inf = pcall(V.lp_info)
if ok then P.info = inf end P.info_t = now end local inf = P.info or { name = V.my_name, dn = V.my_name, id = 0, age = 0 } local kills = V.stats.kills local ri = V.rank_of(kills) local R = V.RANKS[ri]
local nxt = V.RANKS[ri + 1] local rc = R[3] V.rect(o.sh, px - 4, py - 3, W + 8, H + 8, C.shadow); V.alpha(o.sh, 0.4 * k) V.rect(o.bg, px, py, W, H, C.win, C.cardb); V.alpha(o.bg, 0.98 * k)
V.rect(o.ln, px + 10, py, W - 20, 2, rc); V.grad(o.ln, V.lighten(rc, 0.4), rc); V.alpha(o.ln, k) local pulse = 0.5 + (0.5 * math.sin(now * 2.4))
V.rect(o.ring, px + 14, py + 16, 48, 48, rc); V.alpha(o.ring, (0.55 + (0.45 * pulse)) * k) V.rect(o.av, px + 17, py + 19, 42, 42, A); V.grad(o.av, V.lighten(A, 0.35), A); V.alpha(o.av, k)
V.text(o.avt, px + 38, py + 29, string.upper(string.sub(inf.dn ~= "" and inf.dn or inf.name, 1, 1)), V.C_WHITE); V.alpha(o.avt, k) local dn = inf.dn if string.len(dn) > 16 then dn = string.sub(dn, 1, 15) .. ".." end
V.text(o.nm, px + 74, py + 18, dn, V.C_WHITE); V.alpha(o.nm, k) V.text(o.un, px + 74, py + 37, "@" .. inf.name, C.dim); V.alpha(o.un, k)
V.text(o.id, px + 74, py + 53, "ID " .. tostring(inf.id) .. "  ·  " .. tostring(inf.age) .. " days", C.dim); V.alpha(o.id, k) local rw = V.tw(R[2], 11) + 18
V.rect(o.rk, px + 14, py + 76, rw, 20, V.mix(C.win, rc, 0.3), rc); V.alpha(o.rk, k) V.text(o.rkt, (px + 14) + (rw / 2), py + 79, R[2], V.lighten(rc, 0.3)); V.alpha(o.rkt, k) local bx = (px + 24) + rw
local bw = ((px + W) - 14) - bx local f = 1 local nt = "max rank" if nxt then f = (kills - R[1]) / math.max(1, nxt[1] - R[1]) nt = tostring(nxt[1] - kills) .. " kills to " .. string.lower(nxt[2]) end
V.rect(o.bb, bx, py + 80, bw, 5, C.off); V.alpha(o.bb, k) V.rect(o.bf, bx, py + 80, math.max(3, bw * math.max(0, math.min(1, f))), 5, rc); V.alpha(o.bf, k) V.text(o.nx, bx, py + 88, nt, C.dim); V.alpha(o.nx, k)
V.rect(o.cell, px + 14, py + 112, W - 28, 92, C.card, C.cardb); V.alpha(o.cell, k) local deaths = tonumber(V.lp_attr("Deaths")) or V.stats.deaths or 0 local shots = V.stats.hits + V.stats.misses
local hr = (shots > 0) and math.floor(((V.stats.hits / shots) * 100) + 0.5) or 0 local mins = math.floor((now - V.SESSION_T) / 60) local kd = (deaths > 0) and string.format("%.2f", kills / deaths) or tostring(kills)
local labs = { "KILLS", "DEATHS", "K/D", "HIT RATE", "BEST STREAK", "SESSION" } local vals = { tostring(kills), tostring(deaths), kd, tostring(hr) .. "%", tostring(V.stats.best_streak), tostring(mins) .. " min" }
local cw = (W - 28) / 3 for i = 1, 6 do local col = (i - 1) % 3 local row = math.floor((i - 1) / 3) local cx = (px + 14) + (col * cw) + 12 local cy = (py + 120) + (row * 44)
V.text(V.pf_sl[i], cx, cy, labs[i], C.dim); V.alpha(V.pf_sl[i], k) V.text(V.pf_sv[i], cx, cy + 14, vals[i], (i == 1) and A or V.C_WHITE); V.alpha(V.pf_sv[i], k) end
V.rect(V.pf_sep1, (px + 14) + cw, py + 120, 1, 76, C.cardb); V.alpha(V.pf_sep1, k) V.rect(V.pf_sep2, (px + 14) + (cw * 2), py + 120, 1, 76, C.cardb); V.alpha(V.pf_sep2, k) local king = V.EGG.king > 0
local ft = "NeverPeek 1.2  ·  " .. (king and "long live the king" or "stay unpeekable") V.text(o.ft, px + 14, py + 214, ft, king and color(255, 196, 60) or C.dim); V.alpha(o.ft, k) end V.BURST = { list = {}, t = -10 }
V.BURST_N = _v(14) for i = 1, V.BURST_N do V.BURST.list[i] = { d = V.mk_text(18, false, true, 70), on = false } end V.burst = function(x, y, glyph) local now = V.now_s() V.BURST.t = now for i = 1, V.BURST_N do
local b = V.BURST.list[i] local ang = ((i / V.BURST_N) * math.pi * 2) + (V.rnd01() * 0.4) local sp = 160 + (V.rnd01() * 220) b.x = x b.y = y b.vx = math.cos(ang) * sp b.vy = (math.sin(ang) * sp) - 120 b.on = true
b.g = glyph or "🐝" b.sz = 14 + math.floor(V.rnd01() * 10) end end V.draw_burst = function(now, dt) local age = now - V.BURST.t for i = 1, V.BURST_N do local b = V.BURST.list[i] if b.on and age < 1.4 then
b.vy = b.vy + (420 * dt) b.x = b.x + (b.vx * dt) b.y = b.y + (b.vy * dt) if b.d.Size ~= b.sz then b.d.Size = b.sz end V.text(b.d, b.x, b.y, b.g, V.C_WHITE) V.alpha(b.d, 1 - V.ease((age - 0.8) / 0.6)) else
if b.on then b.on = false end V.hide(b.d) end end end V.egg_banner = function(text, chip, sub, label, col, rb) V.AN.text = text V.AN.t = V.now_s() V.AN.hs = false V.AN.n = 1 V.AN.victim = "" V.AN.chip = chip
V.AN.sub = sub V.AN.kind = "egg" V.AN.label = label or "SECRET" V.AN.col = col or false V.AN.rb = rb and true or false
if V.ann_sound:get() ~= "Off" then V.api("sounds.play", sounds and sounds.play, V.ann_sound:get(), 1) end end V.egg_click = function(now, cx, cy) local c = V.EGG.clicks c[#c + 1] = now
while #c > 0 and (now - c[1]) > 1.6 do table.remove(c, 1) end if #c >= 5 then V.EGG.clicks = {} V.EGG.honey = not V.EGG.honey V.burst(cx, cy, "🐝") if V.EGG.honey then
V.egg_banner("HONEY MODE", "🍯", "the hive welcomes you", "SECRET UNLOCKED", V.HONEY_COL) else V.egg_banner("HONEY MODE OFF", "🐝", "back to work, bee", "HIVE CLOSED", color(200, 160, 90)) end return true end return false
end V.egg_key = function(k) if string.len(k) ~= 1 then return end V.EGG.buf = string.sub(V.EGG.buf .. k, -9) if V.EGG.buf == "NEVERPEEK" then V.EGG.buf = "" V.EGG.rainbow_until = V.now_s() + 15
V.burst(V.sw / 2, V.sh / 2, "✨") V.egg_banner("YOU FOUND IT", "★", "never peek. never miss.", "SECRET UNLOCKED", false, true) end end V.CROWN = { t = -10 } V.cr_emo = V.mk_text(30, false, true, 66)
V.cr_bg = V.mk_rect(6, 65) V.cr_ln = V.mk_rect(1, 66) V.cr_tx = V.mk_text(11, false, true, 67, true) V.cr_sp = { V.mk_text(12, false, true, 67), V.mk_text(10, false, true, 67), V.mk_text(9, false, true, 67) }
V.CROWN_LV = { { 5, "KING", color(255, 196, 60) }, { 8, "EMPEROR", color(255, 110, 70) }, { 10, "LEGEND", color(170, 120, 255) }, } V.crown_hide = function()
V.hide(V.cr_emo); V.hide(V.cr_bg); V.hide(V.cr_ln); V.hide(V.cr_tx) V.hide(V.cr_sp[1]); V.hide(V.cr_sp[2]); V.hide(V.cr_sp[3]) end V.crown_level = function(streak) local lv = 0 for i, c in ipairs(V.CROWN_LV) do
if streak >= c[1] then lv = i end end return lv end V.crown_head = function() local ch = V.find_lp().Character if not ch then return false end local h = ch:FindFirstChild("Head") if not h then return false end
return h.Position end V.draw_crown = function(now) local lv = V.crown_level(V.stats.streak) if lv ~= V.EGG.king then if lv > V.EGG.king and lv > 0 then local c = V.CROWN_LV[lv]
V.egg_banner(c[2], "👑", "streak " .. tostring(V.stats.streak) .. "  ·  all hail", "CROWNED", c[3]) V.CROWN.t = now end V.EGG.king = lv end if lv == 0 then if V.cr_emo.Visible then V.crown_hide() end return end
local okh, hp = pcall(V.crown_head) local okc, cp = pcall(V.bee_cam) if not (okh and hp and okc and cp) then V.crown_hide() return end local d = (hp - cp).Magnitude if d < 2 then V.crown_hide() return end
local bob = 0.15 * math.sin(now * 2.2) local x, y = V.bee_proj(hp.X, (hp.Y + 1.9) + bob, hp.Z) if not x then V.crown_hide() return end local sc = math.max(0.45, math.min(1.6, 12 / math.max(3, d)))
local pop = V.ease_back((now - V.CROWN.t) / 0.5) local c = V.CROWN_LV[lv] local es = math.floor(30 * sc * math.max(0.2, pop)) if V.cr_emo.Size ~= es then V.cr_emo.Size = es end V.text(V.cr_emo, x, y - es, "👑", V.C_WHITE)
local ts = 11 local tw = V.tw(c[2], ts) + 16 local ty = y + 2 V.rect(V.cr_bg, x - (tw / 2), ty, tw, 17, color(14, 12, 8), c[3]) V.alpha(V.cr_bg, 0.9) V.rect(V.cr_ln, (x - (tw / 2)) + 4, ty, tw - 8, 2, c[3])
V.text(V.cr_tx, x, ty + 2, c[2], c[3]) for i = 1, 3 do local a = (now * (1.4 + (i * 0.3))) + (i * 2.1) local r = (es * 0.75) + (i * 3) local sx = x + (math.cos(a) * r)
local sy = (y - (es * 0.55)) + (math.sin(a) * r * 0.45) V.text(V.cr_sp[i], sx, sy, "✦", V.lighten(c[3], 0.4)) V.alpha(V.cr_sp[i], 0.4 + (0.6 * math.abs(math.sin((now * 3) + i)))) end end
V.atan2 = math.atan2 or function(y, x) return math.atan(y, x) end V.SB = { list = {}, on = false } V.SB_N = _v(9) V.sb_make = function(i)
local b = { d = V.mk_text(20, false, true, 68), tr = { V.mk_rect(2, 67), V.mk_rect(2, 67) }, hist = {}, ht = -10 } b.x = V.rnd01() * V.sw b.y = V.rnd01() * V.sh b.a = V.rnd01() * math.pi * 2 b.sp = 70 + (V.rnd01() * 70)
b.ph = V.rnd01() * 10 b.sz = 16 + math.floor(V.rnd01() * 10) return b end for i = 1, V.SB_N do V.SB.list[i] = V.sb_make(i) end V.sb_hide = function() for i = 1, V.SB_N do local b = V.SB.list[i]
V.hide(b.d); V.hide(b.tr[1]); V.hide(b.tr[2]) b.hist = {} end V.SB.on = false end V.sb_step = function(b, i, now, dt) local turn = (math.sin((now * 0.9) + b.ph) * 1.6) + (math.sin((now * 2.3) + (b.ph * 2)) * 0.7)
local cx = V.sw / 2 local cy = V.sh / 2 local dx = b.x - cx local dy = b.y - cy local dc = math.sqrt((dx * dx) + (dy * dy)) if dc < 170 then local want = V.atan2(dy, dx)
local diff = ((want - b.a) + math.pi) % (math.pi * 2) turn = turn + ((diff - math.pi) * 3) end local m = 70 if b.x < m or b.x > (V.sw - m) or b.y < m or b.y > (V.sh - m) then local want = V.atan2(cy - b.y, cx - b.x)
local diff = ((want - b.a) + math.pi) % (math.pi * 2) turn = turn + ((diff - math.pi) * 4) end b.a = b.a + (turn * dt) b.x = b.x + (math.cos(b.a) * b.sp * dt) b.y = b.y + (math.sin(b.a) * b.sp * dt)
local bob = math.sin((now * 9) + b.ph) * 2 if b.d.Size ~= b.sz then b.d.Size = b.sz end V.text(b.d, b.x, (b.y + bob) - (b.sz / 2), "🐝", V.C_WHITE) V.alpha(b.d, 0.9) if (now - b.ht) > 0.06 then b.ht = now
b.hist[2] = b.hist[1] b.hist[1] = { x = b.x, y = b.y + bob } end for k = 1, 2 do local p = b.hist[k] if p then local sz = 3 - k V.rect(b.tr[k], p.x - 1, p.y + 2, sz + 1, sz + 1, V.BEE_POL) V.alpha(b.tr[k], 0.55 / k) else
V.hide(b.tr[k]) end end end V.draw_screen_bees = function(now, dt) if not V.EGG.honey then if V.SB.on then V.sb_hide() end return end V.SB.on = true for i = 1, V.SB_N do V.sb_step(V.SB.list[i], i, now, dt) end end
client.set_event_callback("paint", function() local now = V.now_s() local dt = math.max(0, math.min(0.05, now - (V.EGG.ft or now))) V.EGG.ft = now local ok3, e3 = pcall(V.draw_screen_bees, now, dt)
if not ok3 then V.report("screen bees", e3) end local ok1, e1 = pcall(V.draw_burst, now, dt) if not ok1 then V.report("burst", e1) end local ok2, e2 = pcall(V.draw_crown, now) if not ok2 then V.report("crown", e2) end
end) V.GN = { G = 23.48, FULL = 62, MED = 48, SHORT = 35, INH = 1.25, CAP = 35, R = 0.35, DRAG = 0.04, FUSE = 1.8, AIR = 2, DT = 0.015625, BIAS = 0.13333, FWD = 1.25, DOWN = 0.2, TAN = 0.65, NOR = 1.1,
GY = 0.7, SETTLE = 3.24, SLEEP = 0.36, SLIDE = 0.3, MOLY = 0.65, EYE = 4.7, DUCK = 1.4, VMAX = 130 }
V.GS = { cache = false, last = -10, cur = {}, tgt = {}, alpha = 0, smO = false, smP = 0, smY = 0, smDet = false, smGround = false,
tracked = {}, fires = {}, waves = {}, fireOwner = {}, lastScan = -10, t = 0, clip = {}, clipN = 0, lbl_used = 0, lbl_last = 0, err = 0, prm = false, prm_t = -10 } V.GN_N = _v(34) V.GV0 = Vector3.new(0, 0, 0) V.GC = {
he = Color3.fromRGB(90, 220, 255), he_deep = Color3.fromRGB(40, 110, 170), mol = Color3.fromRGB(255, 95, 55), mol_deep = Color3.fromRGB(140, 40, 24),
ash = Color3.fromRGB(110, 60, 50), hot = Color3.fromRGB(255, 60, 60), txt = Color3.fromRGB(238, 240, 246), }
V.gc_mix = function(a, b, t) return Color3.new(a.R + ((b.R - a.R) * t), a.G + ((b.G - a.G) * t), a.B + ((b.B - a.B) * t)) end V.g_damp = function(a, b, k, dt) return a + ((b - a) * (1 - math.exp(-k * dt))) end
V.g_dampA = function(a, b, k, dt) local d = (((b - a) + math.pi) % (2 * math.pi)) - math.pi return a + (d * (1 - math.exp(-k * dt))) end V.g_clamp = function(x, a, b) return math.max(a, math.min(b, x)) end
V.g_easeout = function(t) local k = 1 - t return 1 - ((k * k) * k) end V.g_lbl = {} V.GL_N = _v(20) for i = 1, V.GL_N do V.g_lbl[i] = V.mk_text(13, false, true, 64, true) end V.g_text = function(wp, str, col, size)
local S = V.GS if S.lbl_used >= V.GL_N then return end local ok, x, y, on = pcall(render.world_to_screen, wp) if not (ok and on and type(x) == "number") then return end S.lbl_used = S.lbl_used + 1
local d = V.g_lbl[S.lbl_used] if d.Size ~= size then d.Size = size end V.text(d, x, y - (size / 2), str, col) V.alpha(d, 1) end
V.g_c3 = function(c) return color(math.floor(c.R * 255), math.floor(c.G * 255), math.floor(c.B * 255)) end V.c_r = function(c) return c.R end V.to_c3 = function(c) local ok, r = pcall(V.c_r, c)
if ok and type(r) == "number" then return Color3.new(c.R, c.G, c.B) end return Color3.fromRGB(c.r, c.g, c.b) end V.g_name = function(o) return o.Name end V.g_parent = function(o) return o.Parent end
V.g_attr = function(o, k) return o:GetAttribute(k) end V.g_isclip = function(inst) local S = V.GS if not inst then return false end local c = S.clip[inst] if c ~= nil then return c end local node = inst local res = false
local hops = 0 while node and node ~= workspace and hops < 12 do hops = hops + 1 local okn, n = pcall(V.g_name, node) local oka, a = pcall(V.g_attr, node, "Clip")
if (okn and n and string.find(string.lower(n), "clip", 1, true)) or (oka and a == true) then res = true break end local okp, par = pcall(V.g_parent, node) node = okp and par or nil end
if S.clipN > 400 then S.clip = {} S.clipN = 0 end S.clip[inst] = res S.clipN = S.clipN + 1 return res end V.g_ray = function(o, d, prm) return workspace:Raycast(o, d, prm) end V.g_trace = function(o, d, prm)
local ok, hit = pcall(V.g_ray, o, d, prm) if not ok or not hit then return nil end if not V.g_isclip(hit.Instance) then return hit end local mag = d.Magnitude if mag < 0.0001 then return nil end
local unit = d * (1 / mag) local from = hit.Position + (unit * 0.02) local left = mag - ((hit.Position - o).Magnitude + 0.02) for _ = 1, 7 do if left <= 0.0001 then return nil end
local ok2, nxt = pcall(V.g_ray, from, unit * left, prm) if not ok2 or not nxt then return nil end if not V.g_isclip(nxt.Instance) then return nxt end local used = (nxt.Position - from).Magnitude + 0.02
if left <= used then return nil end from = nxt.Position + (unit * 0.02) left = left - used end return nil end V.g_launch = function(o, pitch, yaw, spd, inh) local P = V.GN local d = math.deg(pitch)
local adj = math.rad(d + ((90 - math.abs(d)) * P.BIAS)) local cp = math.cos(adj) local dir = Vector3.new(-math.sin(yaw) * cp, math.sin(adj), -math.cos(yaw) * cp) local iv = inh or V.GV0
if iv.Magnitude > P.CAP then iv = iv.Unit * P.CAP end return { p = (o + (dir * P.FWD)) - Vector3.new(0, P.DOWN, 0), v = (dir * spd) + (iv * P.INH) } end V.g_step = function(pos, vel, prm) local P = V.GN local dt = P.DT
local k = V.g_clamp(1 - (dt * P.DRAG), 0, 1) local nv = Vector3.new(vel.X * k, (vel.Y - (dt * P.G)) * k, vel.Z * k) local delta = nv * dt local mag = delta.Magnitude
if mag < 0.0001 then return { p = pos, v = nv, rest = true, hit = false } end local hit = V.g_trace(pos, delta * (1 + (P.R / mag)), prm) if not hit then return { p = pos + delta, v = nv, rest = false, hit = false } end
local n = hit.Normal local vn = nv:Dot(n) local rest = false if vn < 0 then nv = (nv * P.TAN) - (n * (vn * P.NOR)) if n.Y >= P.GY then local sq = nv:Dot(nv) if sq < P.SETTLE then if sq < P.SLEEP then nv = V.GV0
rest = true else nv = (nv - (n * nv:Dot(n))) * P.SLIDE end end end end return { p = hit.Position + (n * P.R), v = nv, rest = rest, hit = hit } end V.g_sim = function(sp, sv, maxT, mol, prm) local P = V.GN
local pts = { sp } local n = 1 local t = 0 local rested = false local det = false local pos = sp local vel = sv while t < maxT do t = t + P.DT local r = V.g_step(pos, vel, prm) pos = r.p
if r.hit and mol and r.hit.Normal.Y >= P.MOLY then pos = r.hit.Position det = true n = n + 1 pts[n] = pos break end n = n + 1 pts[n] = pos vel = r.v if r.rest and not mol then rested = true break end end
return { pts = pts, n = n, fin = pos, det = det, t = t, rested = rested } end V.g_resample = function(pts, n, out, count, i0) i0 = i0 or 1 if i0 >= n then for i = 1, count do out[i] = pts[n] end return end local cum = {}
local total = 0 cum[i0] = 0 for i = i0 + 1, n do total = total + (pts[i] - pts[i - 1]).Magnitude cum[i] = total end if total < 0.0001 then for i = 1, count do out[i] = pts[n] end return end local seg = i0 + 1
for i = 1, count do local d = (total * (i - 1)) / (count - 1) while seg < n and cum[seg] < d do seg = seg + 1 end local a = pts[seg - 1] local b = pts[seg] local l0 = cum[seg - 1] local l1 = cum[seg]
local t = ((l1 - l0) > 0.000001) and ((d - l0) / (l1 - l0)) or 0 out[i] = a + ((b - a) * t) end end V.g_morph = function(cur, tgt, count, f) for i = 1, count do local c = cur[i]
if not c then cur[i] = tgt[i] else cur[i] = c + ((tgt[i] - c) * f) end end end V.g_ground = function(p, prm) local ok, hit = pcall(V.g_ray, p + Vector3.new(0, 3, 0), Vector3.new(0, -500, 0), prm)
if not ok or not hit then return { p = p, n = false, d = 999 } end return { p = hit.Position, n = hit.Normal, d = (p - hit.Position).Magnitude } end V.g_safe = function(sp, prm) local g = V.g_ground(sp, prm)
if g.n and sp.Y < (g.p.Y + 0.4) then return Vector3.new(sp.X, g.p.Y + 0.4, sp.Z) end return sp end V.g_params = function() local S = V.GS local now = V.now_s() if S.prm and (now - S.prm_t) < 1 then return S.prm end
local rp = RaycastParams.new() pcall(V.set_prop, rp, "IgnoreWater", true) pcall(V.set_prop, rp, "RespectCanCollide", true) rp.FilterType = Enum.RaycastFilterType.Exclude local f = {}
local pl = game:GetService("Players"):GetPlayers() for _, p in ipairs(pl) do local c = p.Character if c then f[#f + 1] = c end end rp.FilterDescendantsInstances = f S.prm = rp S.prm_t = now return rp end
V.g_equipped = function() local ok, w = pcall(client.get_weapon) if not ok or type(w) ~= "table" then return false end local id = tostring(w.id or "") local nm = tostring(w.name or "")
if id == "Molotov" or nm == "Molotov" then return "mol" end if id == "HE" or nm == "HE Grenade" then return "he" end if w.is_grenade == true or w.category == "Grenade" then local low = string.lower(id .. " " .. nm)
if string.find(low, "molotov", 1, true) or string.find(low, "incendiary", 1, true) then return "mol" end
if string.find(low, "smoke", 1, true) or string.find(low, "flash", 1, true) or string.find(low, "decoy", 1, true) then return "other" end return "he" end return false end V.g_speed = function() local P = V.GN
local m = V.gn_throw:get() if m == "Full" then return P.FULL end if m == "Medium" then return P.MED end if m == "Short" then return P.SHORT end local okl, l = pcall(input.is_mouse_down, 1)
local okr, r = pcall(input.is_mouse_down, 2) local L = okl and l == true local Rr = okr and r == true if L and Rr then return P.MED end if Rr then return P.SHORT end return P.FULL end V.g_lp_pos = function()
local lp = entity.get_local_player() return entity.get_player_position(lp) end V.g_duck = function() return movement.get_duck_amount() end V.g_origin = function() local okr, root = pcall(V.g_lp_pos)
if not okr or not root then return false end local okd, duck = pcall(V.g_duck) local dk = (okd and type(duck) == "number") and duck or 0 return root + Vector3.new(0, V.GN.EYE - (V.g_clamp(dk, 0, 1) * V.GN.DUCK), 0) end
V.g_cam_angles = function() local look = workspace.CurrentCamera.CFrame.LookVector return { p = math.asin(V.g_clamp(look.Y, -1, 1)), y = V.atan2(-look.X, -look.Z) } end V.g_angles = function()
local okc, a = pcall(V.g_cam_angles) if okc and a then return a end local ok, p, y = pcall(client.get_pitch_yaw) if ok and type(p) == "number" and type(y) == "number" then return { p = p, y = y } end
return { p = 0, y = 0 } end V.g_inherit = function() if not V.gn_inh:get() then return V.GV0 end local ok, v = pcall(movement.get_velocity) if ok and v and type(v) ~= "number" then return v end return V.GV0 end
V.g_foes = function() local out = {} local ok, en = V.api("entity.get_enemies", entity.get_enemies) if not ok or type(en) ~= "table" then return out end for _, e in ipairs(en) do
local okp, p = pcall(entity.get_player_position, e) local oka, al = pcall(entity.is_alive, e) if okp and p and (not oka or al ~= false) then local s = V.st(e)
local okh, head = pcall(entity.get_player_hitbox_position, e, "head") local okar, ar = pcall(entity.get_player_armor, e) out[#out + 1] = { p = p, hp = (s and tonumber(s.hp)) or 100, ar = (okar and tonumber(ar)) or 0,
head = (okh and head) or (p + Vector3.new(0, 5, 0)), dmg = 0, fire = false, ttk = 0 } end end return out end V.g_he_dmg = function(dist, ar) local R = V.gn_he_r:get() if dist > R then return 0 end
local dmg = V.gn_he_d:get() * (1 - (dist / R)) if ar and ar > 0 then local quarter = (dmg * 0.5) * 0.5 if ar < quarter then dmg = dmg - (ar * 2) else dmg = dmg * 0.5 end end return math.max(0, math.floor(dmg)) end
V.g_zone_hits = function(foes, pos, mol, remain) local hits = 0 local kills = 0 local R = mol and V.gn_fire_r:get() or V.gn_he_r:get() for _, f in ipairs(foes) do local d = (f.p - pos).Magnitude if d <= R then
if mol then local dps = 40 local ttk = f.hp / dps hits = hits + 1 if ttk <= remain then kills = kills + 1 end f.fire = true f.ttk = ttk f.dmg = dps else local dmg = V.g_he_dmg(d, f.ar) if dmg > 0 then hits = hits + 1
if dmg >= f.hp then kills = kills + 1 end if not f.fire and dmg > f.dmg then f.dmg = dmg end end end end end return { h = hits, k = kills } end
V.g_flat = function(p) return CFrame.new(p) * CFrame.Angles(math.pi * 0.5, 0, 0) end V.g_line = function(a, b, col, th, tr, top) render3d.line(a, a + (a - b), col, th, tr, top) end
V.g_ring = function(p, r, th, col, tr, top) render3d.ring(p, r, th, col, tr, top) end V.g_sphere = function(p, r, col, tr, top) render3d.sphere(p, r, col, tr, top) end
V.g_cyl = function(cf, r, h, col, tr, top) render3d.cylinder(cf, r, h, col, tr, top) end V.g_arc = function(pts, count, colA, colB, th, tr, top, reveal) local lim = math.max(2, math.floor(count * reveal))
for i = 1, lim - 1 do local a = pts[i] local b = pts[i + 1] if a and b then pcall(V.g_line, a, b, V.gc_mix(colA, colB, (i - 1) / (count - 1)), th, tr, top) end end end
V.g_zone = function(zp, R, col, tr, top, now, lethal) pcall(V.g_cyl, V.g_flat(zp), R, 0.04, col, 0.9, top) pcall(V.g_ring, zp, R, 0.2, col, tr, top) local f = (now * 0.5) % 1
pcall(V.g_ring, zp + Vector3.new(0, 0.03, 0), R * V.g_easeout(f), 0.12, col, V.g_clamp(tr + (f * 0.6), 0, 0.97), top) if lethal then local p = 0.5 + (0.5 * math.sin(now * 5))
pcall(V.g_ring, zp + Vector3.new(0, 0.05, 0), R * (0.97 + (0.03 * p)), 0.1, V.GC.txt, V.g_clamp(0.3 + (0.35 * p), 0, 0.9), top) end end V.G_NADE_KEYS = { "grenade", "molotov", "incendiary", "frag", "nade" }
V.G_FIRE_KEYS = { "fire", "flame", "inferno", "burn" } V.G_CONT = { "grenade", "throwable", "projectile", "effect", "debris", "temp", "client", "nade", "fire" } V.g_has = function(low, keys) for _, k in ipairs(keys) do
if string.find(low, k, 1, true) then return true end end return false end V.g_kind = function(nm) local low = string.lower(nm) if low == "he" or low == "he grenade" then return "he" end
if low == "molotov" then return "mol" end if V.g_has(low, V.G_NADE_KEYS) then if string.find(low, "molotov", 1, true) or string.find(low, "incendiary", 1, true) then return "mol" end
if string.find(low, "smoke", 1, true) or string.find(low, "flash", 1, true) then return false end return "he" end return false end V.g_part = function(o) if o:IsA("BasePart") then return o end
if o.PrimaryPart then return o.PrimaryPart end return o:FindFirstChildWhichIsA("BasePart") end V.g_owned = function(o) local node = o local hops = 0 local plrs = game:GetService("Players")
while node and node ~= workspace and hops < 10 do local nm = node.Name local low = string.lower(nm)
if nm == "Characters" or low == "viewmodel" or low == "arms" or low == "equippedweapon" or low == "handle" then return true end if plrs:FindFirstChild(nm) then return true end if node:IsA("Tool") then return true end
if node:FindFirstChildOfClass("Humanoid") then return true end node = node.Parent hops = hops + 1 end return false end V.g_welded = function(part)
return (part:FindFirstChildWhichIsA("WeldConstraint") or part:FindFirstChildWhichIsA("Weld") or part:FindFirstChildWhichIsA("Motor6D")) ~= nil end V.g_root = function(o) local best = o local node = o local hops = 0
while node and hops < 4 do local par = node.Parent if not par or par == workspace then break end if V.g_kind(par.Name) and par:IsA("Model") then best = par node = par hops = hops + 1 else break end end return best end
V.g_add_fire = function(pos, now, obj) local S = V.GS if obj and S.fireOwner[obj] then return S.fireOwner[obj] end local R = math.max(12, V.gn_fire_r:get() * 1.6) local z = false for _, f in ipairs(S.fires) do
if (f.pos - pos).Magnitude <= R then z = f break end end if not z then if #S.fires >= 8 then return false end
z = { pos = pos, sm = pos, t0 = now, dur = V.gn_fire_t:get(), parts = {}, real = false, seed = (now * 13) % 6.28 } S.fires[#S.fires + 1] = z end if obj and #z.parts < 20 then local okp, part = pcall(V.g_part, obj)
if okp and part then z.parts[#z.parts + 1] = { o = obj, part = part } z.real = true S.fireOwner[obj] = z end end return z end V.g_consider = function(o, now, myPos) local S = V.GS local nm = o.Name
local low = string.lower(nm) if V.gn_fire:get() and V.g_has(low, V.G_FIRE_KEYS) then if S.fireOwner[o] then return end local part = V.g_part(o) if not part then return end local p = part.Position
if (p - myPos).Magnitude <= 260 then V.g_add_fire(p, now, o) end return end local kind = V.g_kind(nm) if not kind then local oka, a = pcall(V.g_attr, o, "GrenadeType")
if oka and type(a) == "string" then kind = V.g_kind(a) end end if not kind then return end local root = V.g_root(o) if S.tracked[root] then return end local part = V.g_part(root) if not part then return end
local pos = part.Position if (pos - myPos).Magnitude > 260 then return end if V.g_owned(root) or V.g_owned(part) or V.g_welded(part) then return end
S.tracked[root] = { obj = root, part = part, kind = kind, pos = pos, sPos = pos, sT = now, vs = V.GV0, hasV = false,
born = now, lt = false, reveal = 0, cur = {}, tgt = {}, n = 18, trail = {}, trailT = 0, launched = false, t0 = 0, full = false, land = false, smLand = false, onGround = false, refits = 0 } end
V.g_scan = function(now, myPos) local kids = workspace:GetChildren() local budget = 240 for _, o in ipairs(kids) do if budget <= 0 then break end budget = budget - 1 pcall(V.g_consider, o, now, myPos)
if o:IsA("Folder") and V.g_has(string.lower(o.Name), V.G_CONT) then for _, c in ipairs(o:GetChildren()) do if budget <= 0 then break end budget = budget - 1 pcall(V.g_consider, c, now, myPos) end end end end
V.g_capture = function(e, pos, vel, now, prm) local mol = e.kind == "mol" local sp = V.g_safe(pos, prm) e.t0 = now local flown = now - (e.lt or now)
e.full = V.g_sim(sp, vel, math.max(0.1, (mol and V.GN.AIR or V.GN.FUSE) - flown), mol, prm) local g = V.g_ground(e.full.fin, prm) e.land = g.p
e.onGround = (g.d <= 1.8 and g.n and g.n.Y >= 0.5) or e.full.rested or e.full.det end V.g_wave = function(at, now, onGround) local S = V.GS
if #S.waves < 5 then S.waves[#S.waves + 1] = { p = at, t0 = now, g = onGround } end end V.g_part_alive = function(e) return e.part.Parent ~= nil end V.g_live = function(now, dt, myPos, prm, foes, top, A3) local S = V.GS
for o, e in pairs(S.tracked) do local oka, alive = pcall(V.g_part_alive, e) local pos = false if oka and alive then pos = e.part.Position end if not pos or (now - e.born) > 10 then if e.launched then
local at = e.smLand or e.pos if e.kind == "mol" then if V.gn_fire:get() then V.g_add_fire(at, now, false) end else V.g_wave(at, now, e.onGround) end end S.tracked[o] = nil else e.pos = pos local span = now - e.sT
if span >= (1 / 30) then local v = (pos - e.sPos) * (1 / span) local raw = e.part.AssemblyLinearVelocity if raw.Magnitude >= 3 and raw.Magnitude <= V.GN.VMAX then
if v.Magnitude < 1 or (raw - v).Magnitude < (raw.Magnitude * 0.6) then v = raw end end if v.Magnitude <= V.GN.VMAX then if not e.hasV then e.vs = v e.hasV = true else e.vs = V.g_damp(e.vs, v, 18, span) end end
e.sPos = pos e.sT = now end if not e.launched then if e.hasV and e.vs.Magnitude >= 10 and (pos - myPos).Magnitude > 2.5 then e.launched = true e.born = now e.lt = now e.trail[1] = pos V.g_capture(e, pos, e.vs, now, prm)
elseif (now - e.born) > 1.5 then S.tracked[o] = nil end end if e.launched and e.full then e.reveal = math.min(1, e.reveal + (dt / 0.25)) if (now - e.trailT) >= 0.045 then e.trailT = now local n = #e.trail
if n == 0 or (pos - e.trail[n]).Magnitude > 0.4 then if n >= 60 then table.remove(e.trail, 1) end e.trail[#e.trail + 1] = pos end end local mol = e.kind == "mol" local age = now - e.t0
local i0 = V.g_clamp(math.floor(age / V.GN.DT) + 1, 1, e.full.n) if e.hasV and e.refits < 6 and age > 0.12 and i0 < e.full.n then if (pos - e.full.pts[i0]).Magnitude > 3 then e.refits = e.refits + 1
V.g_capture(e, pos, e.vs, now, prm) i0 = 1 end end local done = (mol and e.full.det and age >= e.full.t) or ((not mol) and (now - e.lt) >= V.GN.FUSE) if done then local at = e.smLand or e.full.fin if mol then
if V.gn_fire:get() then V.g_add_fire(at, now, false) end else V.g_wave(at, now, e.onGround) end S.tracked[o] = nil else V.g_resample(e.full.pts, e.full.n, e.tgt, e.n, i0) end end end end local mf = 1 - math.exp(-22 * dt)
for _, e in pairs(S.tracked) do if e.launched and e.full then local mol = e.kind == "mol" local age = now - e.t0 local remain = mol and math.max(0, e.full.t - age) or math.max(0, V.GN.FUSE - (now - e.lt))
local col = mol and V.GC.mol or A3 local deep = mol and V.GC.mol_deep or V.gc_mix(A3, Color3.new(0, 0, 0), 0.5) V.g_morph(e.cur, e.tgt, e.n, mf)
if e.land then e.smLand = e.smLand and V.g_damp(e.smLand, e.land, 12, dt) or e.land end local n = #e.trail if n > 1 then for i = 1, n - 1 do
pcall(V.g_line, e.trail[i], e.trail[i + 1], deep, 1.4, V.g_clamp(0.6 - (0.35 * (i / n)), 0, 0.95), top) end end if e.cur[1] then pcall(V.g_line, e.pos, e.cur[1], col, 1.7, 0.2, top)
V.g_arc(e.cur, e.n, col, V.gc_mix(col, V.GC.txt, 0.25), 1.7, 0.3, top, V.g_easeout(e.reveal)) end if e.smLand then local z = V.g_zone_hits(foes, e.smLand, mol, V.gn_fire_t:get())
if (not V.gn_ground:get()) or e.onGround then V.g_zone(e.smLand + Vector3.new(0, 0.05, 0), mol and V.gn_fire_r:get() or V.gn_he_r:get(), col, 0.25, top, now, z.k > 0) end if z.h > 0 then
V.g_text(e.smLand + Vector3.new(0, 1.5, 0), (z.k > 0) and string.format("%d HIT · %d KILL", z.h, z.k) or string.format("%d HIT", z.h), V.g_c3((z.k > 0) and V.GC.hot or col), 12) end end
pcall(V.g_sphere, e.pos, 0.28, col, 0, top) V.g_text(e.pos + Vector3.new(0, 1.4, 0), string.format("%s %.1f", mol and "MOLLY" or "HE", remain), V.g_c3(col), 14) end end end
V.g_fire_alive = function(rec) return rec.o.Parent ~= nil end V.g_fires = function(now, dt, myPos, prm, foes, top) local S = V.GS for i = #S.fires, 1, -1 do local z = S.fires[i] if z.real then local sum = V.GV0
local n = 0 for j = #z.parts, 1, -1 do local rec = z.parts[j] local oka, al = pcall(V.g_fire_alive, rec) if oka and al then sum = sum + rec.part.Position n = n + 1 else S.fireOwner[rec.o] = nil
z.parts[j] = z.parts[#z.parts] z.parts[#z.parts] = nil end end if n > 0 then z.pos = sum * (1 / n) else z.real = false z.t0 = math.min(z.t0, (now - z.dur) + 0.4) end end local remain = z.dur - (now - z.t0)
if remain <= 0 then for _, rec in ipairs(z.parts) do S.fireOwner[rec.o] = nil end S.fires[i] = S.fires[#S.fires] S.fires[#S.fires] = nil else z.sm = V.g_damp(z.sm, z.pos, 10, dt)
local frac = V.g_clamp(remain / z.dur, 0, 1) local fade = V.g_clamp(0.15 + ((1 - math.min(1, remain / 0.6)) * 0.7), 0, 0.95) local col = V.gc_mix(V.GC.ash, V.GC.mol, frac) local g = V.g_ground(z.sm, prm)
local R = V.gn_fire_r:get() local zh = V.g_zone_hits(foes, z.sm, true, remain) V.g_zone(g.p + Vector3.new(0, 0.05, 0), R, col, fade, top, now, zh.k > 0)
pcall(V.g_ring, g.p + Vector3.new(0, 0.07, 0), R * frac, 0.14, col, V.g_clamp(fade + 0.1, 0, 0.92), top) local fl = 0.5 + (0.5 * math.sin((now * 7) + z.seed))
pcall(V.g_ring, g.p + Vector3.new(0, 0.09, 0), R * (0.88 + (0.09 * fl)), 0.08, V.GC.mol, V.g_clamp(0.35 + (0.35 * fl), 0, 0.9), top)
V.g_text(z.sm + Vector3.new(0, 1.5, 0), string.format("FIRE %.1f", remain), V.g_c3(col), 14) end end for i = #S.waves, 1, -1 do local w = S.waves[i] local a = (now - w.t0) / 0.65 if a >= 1 then
S.waves[i] = S.waves[#S.waves] S.waves[#S.waves] = nil else pcall(V.g_ring, w.p + Vector3.new(0, 0.08, 0), 1 + (V.g_easeout(a) * V.gn_he_r:get()), 0.28, V.GC.he, a, top) end end end
V.g_predict = function(now, dt, prm, foes, top, A3) local S = V.GS local kind = V.g_equipped() local nade = (kind == "he" or kind == "mol") local mol = kind == "mol"
S.alpha = V.g_damp(S.alpha, (V.gn_pred:get() and nade) and 1 or 0, 10, dt) if S.alpha < 0.02 or not nade then return end local o = V.g_origin() if not o then return end local an = V.g_angles() local sm = 16
if not S.smO or (o - S.smO).Magnitude > 25 then S.smO = o S.smP = an.p S.smY = an.y else S.smO = V.g_damp(S.smO, o, sm, dt) S.smP = V.g_dampA(S.smP, an.p, sm, dt) S.smY = V.g_dampA(S.smY, an.y, sm, dt) end
local spd = V.g_speed() local c = S.cache local stale = (not c) or (now - S.last) >= (1 / 30) if stale then local L = V.g_launch(S.smO, S.smP, S.smY, spd, V.g_inherit()) local sp = V.g_safe(L.p, prm)
local sim = V.g_sim(sp, L.v, mol and V.GN.AIR or V.GN.FUSE, mol, prm) V.g_resample(sim.pts, sim.n, S.tgt, V.GN_N, 1) local g = V.g_ground(sim.fin, prm)
S.cache = { sim = sim, det = sim.fin, ground = g.p, onGround = (g.d <= 1.8 and g.n and g.n.Y >= 0.5) or sim.rested or sim.det, mol = mol } S.last = now c = S.cache end local mf = 1 - math.exp(-22 * dt)
V.g_morph(S.cur, S.tgt, V.GN_N, mf) S.smDet = S.smDet and V.g_damp(S.smDet, c.det, sm + 4, dt) or c.det S.smGround = S.smGround and V.g_damp(S.smGround, c.ground, sm + 4, dt) or c.ground
local fade = V.g_clamp(0.1 + ((1 - S.alpha) * 0.8), 0, 0.95) local col = mol and V.GC.mol or A3 local deep = mol and V.GC.mol_deep or V.gc_mix(A3, Color3.new(0, 0, 0), 0.45)
V.g_arc(S.cur, V.GN_N, deep, col, 3, math.max(0, fade - 0.05), top, V.g_easeout(S.alpha)) local z = V.g_zone_hits(foes, S.smDet, mol, V.gn_fire_t:get()) if (not V.gn_ground:get()) or c.onGround then
V.g_zone(S.smGround + Vector3.new(0, 0.05, 0), mol and V.gn_fire_r:get() or V.gn_he_r:get(), col, fade, top, now, z.k > 0) end pcall(V.g_sphere, S.smDet, 0.26 + (0.04 * math.sin(now * 4)), col, fade, top)
local tag = mol and string.format("MOLLY  ·  burn %.0fs", V.gn_fire_t:get()) or string.format("HE  ·  %.1fs", V.GN.FUSE) if z.h > 0 then
tag = (z.k > 0) and string.format("%d HIT · %d KILL", z.h, z.k) or string.format("%d HIT", z.h) end V.g_text(S.smDet + Vector3.new(0, 1.6, 0), tag, V.g_c3((z.k > 0) and V.GC.hot or col), 13) end
V.g_labels = function(foes) if not V.gn_lbl:get() then return end for _, f in ipairs(foes) do if f.dmg > 0 then if f.fire then local lethal = f.ttk <= V.gn_fire_t:get()
V.g_text(f.head + Vector3.new(0, 1.1, 0), lethal and string.format("KILL %.1fs", f.ttk) or string.format("-%d/s", math.floor(f.dmg + 0.5)), V.g_c3(lethal and V.GC.hot or V.GC.mol), 13) else local lethal = f.dmg >= f.hp
V.g_text(f.head + Vector3.new(0, 1.1, 0), lethal and "KILL" or string.format("-%d", f.dmg), V.g_c3(lethal and V.GC.hot or V.GC.he), 13) end end end end V.TR = { list = {}, last = false } V.TR_WHITE = Color3.new(1, 1, 1)
V.TR_COL = { pend = Color3.fromRGB(235, 240, 255), hit = false, head = Color3.fromRGB(255, 200, 70), miss = Color3.fromRGB(255, 80, 90) } V.tr_pens = function(from, to, prm) local pens = {} local d = to - from
local total = d.Magnitude if total < 0.5 then return pens end local u = d * (1 / total) local cur = from for _ = 1, 6 do local left = (to - cur).Magnitude if left < 0.2 then break end
local ok, hit = pcall(V.g_ray, cur, u * left, prm) if not ok or not hit then break end pens[#pens + 1] = hit.Position cur = hit.Position + (u * 0.6) end return pens end V.tr_stop = function(e, prm)
local d = e.to - e.from local m = d.Magnitude if m < 0.1 then return e.to end local u = d * (1 / m) local ok, hit = pcall(V.g_ray, e.to + (u * 0.3), u * 400, prm) if ok and hit then return hit.Position end
return e.to + (u * 400) end V.tr_find = function(id) local L = V.TR.list for i = #L, 1, -1 do local e = L[i] if id ~= nil and e.id == id then return e end end for i = #L, 1, -1 do
if L[i].res == "pend" then return L[i] end end return false end V.tr_muzzle = function(o) local cf = workspace.CurrentCamera.CFrame local base = o if (cf.Position - o).Magnitude < 2.5 then base = cf.Position end
return ((base + (cf.RightVector * 0.6)) - (cf.UpVector * 0.5)) + (cf.LookVector * 1.4) end V.tr_fire = function(e) if not V.tr_on:get() then return end local pt = e.point
if pt == nil or type(pt) == "number" then return end local o = V.g_origin() if not o then return end local prm = V.g_params() local okm, mz = pcall(V.tr_muzzle, o) local from = (okm and mz) or o
local en = { from = from, to = pt, stop = pt, t = V.now_s(), id = e.id, res = "pend", dmg = 0, pens = V.tr_pens(o, pt, prm) } local L = V.TR.list L[#L + 1] = en if #L > 12 then table.remove(L, 1) end end
V.tr_hit = function(e) local en = V.tr_find(e.id) if not en then return end en.res = (string.lower(tostring(e.hitbox)) == "head") and "head" or "hit" en.dmg = tonumber(e.damage) or 0 en.rt = V.now_s() end
V.tr_miss = function(e) local en = V.tr_find(e.id) if not en then return end en.res = "miss" en.reason = tostring(e.reason or "?") en.rt = V.now_s() local prm = V.g_params() local okst, st = pcall(V.tr_stop, en, prm)
if okst and st then en.stop = st local more = V.tr_pens(en.to, st, prm) for _, p in ipairs(more) do en.pens[#en.pens + 1] = p end end end client.set_event_callback("aim_fire", function(e) pcall(V.tr_fire, e) end)
client.set_event_callback("aim_hit", function(e) pcall(V.tr_hit, e) end) client.set_event_callback("aim_miss", function(e) pcall(V.tr_miss, e) end) V.tr_draw = function(now, A3) local L = V.TR.list
if #L == 0 then return false end local life = V.tr_time:get() local top = V.gn_xray:get() local drew = false for i = #L, 1, -1 do local e = L[i] local age = now - e.t if age > life then table.remove(L, i) else
drew = true local fade = V.g_clamp((age - (life - 0.5)) / 0.5, 0, 1) local col = V.TR_COL.pend
if e.res == "hit" then col = A3 elseif e.res == "head" then col = V.TR_COL.head elseif e.res == "miss" then col = V.TR_COL.miss end local endp = (e.res == "miss") and e.stop or e.to
local grow = V.g_clamp(age / 0.08, 0, 1) local tip = e.from + ((endp - e.from) * grow) pcall(V.g_line, e.from, tip, col, 5, 0.45 + (0.5 * fade), top)
pcall(V.g_line, e.from, tip, V.gc_mix(col, V.TR_WHITE, 0.55), 2.2, 0.8 * fade, top) if grow >= 1 then for _, p in ipairs(e.pens) do pcall(V.g_sphere, p, 0.13, col, 0.25 + (0.7 * fade), top) end
pcall(V.g_sphere, endp, (e.res == "head") and 0.3 or 0.22, col, 0.1 + (0.85 * fade), top) if (e.res == "hit" or e.res == "head") and e.dmg > 0 then local rise = math.min(1.2, (now - (e.rt or now)) * 1.5)
V.g_text(endp + Vector3.new(0, 0.8 + rise, 0), ((e.res == "head") and "HS  -" or "-") .. tostring(math.floor(e.dmg)), V.g_c3(col), 14) elseif e.res == "miss" then
V.g_text(e.to + Vector3.new(0, 0.8, 0), e.reason or "miss", V.g_c3(col), 12) end end end end return drew end V.g_clear = function() render3d.clear() end V.g_frame = function() local S = V.GS local now = V.now_s()
local dt = V.g_clamp(now - S.t, 0.001, 0.1) S.t = now local any = V.gn_pred:get() or V.gn_live:get() or V.gn_fire:get() if S.drew then pcall(V.g_clear) end S.drew = false for i = 1, S.lbl_last do V.hide(V.g_lbl[i]) end
S.lbl_used = 0 S.lbl_last = 0 local okc0, A30 = pcall(V.to_c3, V.accent_col:get()) if not okc0 or not A30 then A30 = V.GC.he end local okt, tdrew = pcall(V.tr_draw, now, A30) if okt and tdrew then S.drew = true end
S.lbl_last = S.lbl_used if not any then return end local okm, myPos = pcall(V.g_lp_pos) if not okm or not myPos then return end if V.gn_live:get() and (now - S.lastScan) >= 0.25 then S.lastScan = now
pcall(V.g_scan, now, myPos) end local held = V.gn_pred:get() and V.g_equipped() local busy = (held == "he") or (held == "mol") or (next(S.tracked) ~= nil) or (#S.fires > 0) or (#S.waves > 0) or (S.alpha > 0.02)
if not busy then return end S.drew = true local prm = V.g_params() if not S.foes or (now - (S.foes_t or -10)) > 0.1 then S.foes = V.g_foes() S.foes_t = now end local foes = S.foes for _, f in ipairs(foes) do f.dmg = 0
f.fire = false f.ttk = 0 end local top = V.gn_xray:get() local okc, A3 = pcall(V.to_c3, V.accent_col:get()) if not okc or not A3 then A3 = V.GC.he end
if V.gn_live:get() then V.g_live(now, dt, myPos, prm, foes, top, A3) end if V.gn_fire:get() or V.gn_live:get() then V.g_fires(now, dt, myPos, prm, foes, top) end V.g_predict(now, dt, prm, foes, top, A3) V.g_labels(foes)
S.lbl_last = S.lbl_used end client.set_event_callback("paint", function() local p0 = V.pf0() local ok, err = pcall(V.g_frame) V.pf1("grenades", p0) if not ok then V.GS.err = V.GS.err + 1
if V.GS.err <= 3 then V.log(255, 120, 120, "[NP] grenades: " .. tostring(err)) end end end) client.set_event_callback("shutdown", function() pcall(V.g_clear) end) client.set_event_callback("paint", function(dt)
local pm = V.pf0() local okm, em = pcall(V.menu_paint, dt) if not okm then V.report("paint:menu", em) end V.pf1("menu", pm) end) V.menu_paint = function(dt) dt = dt or 0.016 local now = V.now_s()
local since = now - V.ui_open_t if not V.ui_open and since > 0.2 then if V.m_win.Visible then V.m_hide_all() end V.cur_hide() return end local C = V.MC local A = V.accent_col:get() or color(70, 140, 255)
local A2 = V.lighten(A, 0.35) local k = V.ui_open and V.ease(since / 0.22) or (1 - V.ease(since / 0.16)) local oy = (1 - k) * 18 local ks = math.min(1, dt * 14) local okm, mx0, my0 = pcall(input.get_mouse_position)
local good = okm and type(mx0) == "number" and type(my0) == "number" local mx = good and mx0 or -1 local my = good and my0 or -1 local okd, down = pcall(input.is_mouse_down, 1) down = okd and down == true
local pressed = down and not V.m_prev V.m_prev = down local moved = (mx ~= V.m_lmx) or (my ~= V.m_lmy) V.m_lmx = mx V.m_lmy = my
if moved or down or pressed or since < 0.45 or V.m_drag or V.SR.focus or V.m_dd or (now - V.m_tab_t) < 0.35 then V.m_act_t = now end local idle_iv = ((now - V.m_act_t) > 0.6) and 0.1 or (1 / 60)
if not (pressed or down) and (now - V.m_full_t) < idle_iv then if V.ui_open and mx >= 0 then V.draw_dot_cursor(mx, my, A2, V.m_hot_prev) else V.cur_hide() end return end V.m_full_t = now V.m_hot = false
V.m_hover_w = false V.m_hover_tab = false local ovs = V.OV_SR local ovp = V.OV_PF V.m_block = (ovs and V.m_in(mx, my, ovs[1], ovs[2], ovs[3], ovs[4])) or (ovp and V.m_in(mx, my, ovp[1], ovp[2], ovp[3], ovp[4])) or false
local grab_side = V.m_in(mx, my, V.m_x, V.m_y, V.MSIDE, 64) local grab_top = V.m_in(mx, my, V.m_x + V.MSIDE, V.m_y, V.MW - V.MSIDE, V.MTOP) if V.ui_open and pressed and not V.m_dd and (grab_side or grab_top) then
V.m_drag = true V.m_dx = mx - V.m_x V.m_dy = my - V.m_y end local x = V.m_x local y = V.m_y + oy local cx0 = (x + V.MSIDE) + V.MPAD local chip1_hit = V.m_in(mx, my, cx0, y + 12, 132, 30)
local sr_hit = V.m_in(mx, my, cx0 + 144, y + 12, V.SR_W, 30) if V.m_drag then if (chip1_hit or sr_hit) and pressed then V.m_drag = false end end if V.m_drag then if down then
V.m_x = math.floor(math.max(0, math.min(V.sw - V.MW, mx - V.m_dx))) V.m_y = math.floor(math.max(0, math.min(V.sh - V.MH, my - V.m_dy))) else V.m_drag = false end end
V.rect(V.m_shadow, x - 6, y - 4, V.MW + 12, V.MH + 12, C.shadow) V.alpha(V.m_shadow, 0.35 * k) V.rect(V.m_win, x, y, V.MW, V.MH, C.win, C.cardb) V.alpha(V.m_win, 0.94 * k) V.rect(V.m_side, x, y, V.MSIDE, V.MH, C.side)
V.alpha(V.m_side, 0.97 * k) V.rect(V.m_sidefix, (x + V.MSIDE) - 14, y, 14, V.MH, C.side) V.alpha(V.m_sidefix, 0.97 * k) V.rect(V.m_sideln, x + V.MSIDE, y + 12, 1, V.MH - 24, C.cardb)
local lp = 0.5 + (0.5 * math.sin(now * 2.2)) V.rect(V.m_logo_glow, x + 15, y + 14, 36, 36, A) V.alpha(V.m_logo_glow, (0.12 + (0.10 * lp)) * k) V.rect(V.m_logo, x + 18, y + 17, 30, 30, A) V.gradA(V.m_logo, A2, A, 45)
V.rect(V.m_logo_in, x + 20, y + 19, 26, 26, C.side) V.gradA(V.m_logo_in, V.LOGO_G1, V.LOGO_G2, 90) V.text(V.m_logo_t, x + 33, y + 22, "N", A2) V.hide(V.m_logo_dot) V.text(V.m_title, x + 58, y + 17, "Never", V.C_WHITE)
V.text(V.m_title_a, x + 58 + math.floor(V.tw_obj(V.m_title, "Never", 15) + 0.5), y + 17, "Peek", A) V.text(V.m_sub, x + 58, y + 35, "rage lua  ·  1.2", C.dim) local hov_any_tab = false for g = 1, #V.SIDE_GROUPS do
V.G_K[g] = V.m_lerp(V.G_K[g], V.G_OPEN[g] and 1 or 0, math.min(1, dt * 11)) if math.abs(V.G_K[g] - (V.G_OPEN[g] and 1 or 0)) < 0.004 then V.G_K[g] = V.G_OPEN[g] and 1 or 0 end end local yy = 72 local lastg = 0
for r = 1, V.SIDE_N do local row = V.SIDE_ROWS[r] if row.kind == "head" then if lastg > 0 then yy = yy + 8 end lastg = row.g row.dy = yy yy = yy + 22 else row.dy = yy
local hid = V.UI_TABS[row.tab].adv and not V.show_adv:get() row.hid = hid if not hid then yy = yy + (32 * V.ease(V.G_K[row.g])) end end end if V.UI_TABS[V.m_tab] and V.UI_TABS[V.m_tab].adv and not V.show_adv:get() then
V.m_tab = 1 V.m_tab_t = now end for r = 1, V.SIDE_N do local row = V.SIDE_ROWS[r] local ry = y + row.dy if row.kind == "head" then local g = row.g
local hh = V.ui_open and not V.m_dd and not V.m_block and V.m_in(mx, my, x + 12, ry - 2, V.MSIDE - 24, 20) if hh then V.m_hot = true end if hh and pressed then V.G_OPEN[g] = not V.G_OPEN[g] end
V.text(V.m_sh[r], x + 20, ry + 4, row.text, hh and C.text or C.head) local kg = V.G_K[g] local ac = (kg > 0.5) and "v" or ">" V.text(V.m_sarr[r], (x + V.MSIDE) - 30, ry + 2, ac, hh and A or C.head) if not row.clean then
row.clean = true V.hide(V.m_sbg[r]); V.hide(V.m_stx[r]) V.icon_hide(V.m_sic[r]) end else if not row.clean then row.clean = true V.hide(V.m_sh[r]); V.hide(V.m_sarr[r]) end local i = row.tab local kg = V.G_K[row.g]
if row.hid then kg = 0 end if kg < 0.04 then V.hide(V.m_sbg[r]); V.hide(V.m_stx[r]) V.icon_hide(V.m_sic[r]) else local ka = V.ease(kg) local ryy = ry - ((1 - ka) * 10)
local hov = (kg > 0.9) and V.ui_open and not V.m_dd and not V.m_block and V.m_in(mx, my, x + 12, ryy, V.MSIDE - 24, 30) if hov then hov_any_tab = true; V.m_hot = true; V.m_hover_tab = i end
if hov and pressed and V.m_tab ~= i then V.m_tab = i; V.m_tab_t = now; V.m_slider = nil end V.m_tanim[i] = V.m_lerp(V.m_tanim[i], (i == V.m_tab) and 1 or 0, ks) V.m_thov[i] = V.m_lerp(V.m_thov[i], hov and 1 or 0, ks)
local a = V.m_tanim[i] local hv = V.m_thov[i] if a > 0.02 or hv > 0.02 then V.rect(V.m_sbg[r], x + 12, ryy, V.MSIDE - 24, 30, V.mix(C.hov, C.sel, a)) V.alpha(V.m_sbg[r], math.max(a, hv * 0.8) * ka) else
V.hide(V.m_sbg[r]) end local ic = V.mix(V.mix(C.off, C.dim, hv), A, a) V.draw_icon(V.m_sic[r], V.UI_TABS[i].name, x + 22, ryy + 8, ic) V.icon_alpha(V.m_sic[r], ka)
V.text(V.m_stx[r], x + 44, ryy + 7, V.UI_TABS[i].name, V.mix(V.mix(C.dim, C.text, hv), V.C_WHITE, a)) V.alpha(V.m_stx[r], ka) end end end local uy = (y + V.MH) - 62
local uhov = V.ui_open and not V.m_dd and V.m_in(mx, my, x + 12, uy, V.MSIDE - 24, 50) if uhov then V.m_hot = true end if uhov and pressed then
if not V.egg_click(now, x + 37, uy + 25) then V.PF2.open = not V.PF2.open end elseif pressed and V.PF2.open and not V.m_block then V.PF2.open = false end
V.rect(V.m_ubg, x + 12, uy, V.MSIDE - 24, 50, uhov and C.sel or C.card, C.cardb) V.rect(V.m_uav, x + 22, uy + 10, 30, 30, A); V.grad(V.m_uav, A2, A) local nm = (V.my_name ~= "" and V.my_name) or "user"
V.text(V.m_uav_t, x + 37, uy + 17, string.upper(string.sub(nm, 1, 1)), V.C_WHITE) V.text(V.m_un, x + 60, uy + 10, (string.len(nm) > 12) and (string.sub(nm, 1, 11) .. "..") or nm, C.text)
V.text(V.m_us, x + 60, uy + 28, V.k_menu:get() .. " to close", C.dim) V.text(V.m_uch, (x + V.MSIDE) - 30, uy + 17, ">", C.dim) local c1h = chip1_hit and V.ui_open if c1h then V.m_hot = true end
local saved = (now - (V.m_saved_t or -10)) < 1.6 V.rect(V.m_chip1, cx0, y + 12, 132, 30, c1h and C.sel or C.chip, saved and V.C_GOOD or C.cardb) V.rect(V.m_chip1_i, cx0 + 12, y + 22, 10, 10, saved and V.C_GOOD or A)
V.text(V.m_chip1_t, cx0 + 30, y + 19, saved and "Saved" or "Save Code", saved and V.C_GOOD or C.text) V.text(V.m_chip1_a, (cx0 + 132) - 20, y + 19, saved and "ok" or ">", C.dim)
local c2 = tostring(math.floor(V.fps_s + 0.5)) .. " fps  ·  " .. tostring(V.ping_ms or 0) .. " ms" if V.m_c2s ~= c2 then V.m_c2s = c2 V.m_c2w = V.tw(c2, 12) + 24 end local c2w = V.m_c2w
V.rect(V.m_chip2, ((x + V.MW) - V.MPAD) - c2w, y + 12, c2w, 30, C.chip, C.cardb) V.text(V.m_chip2_t, (((x + V.MW) - V.MPAD) - c2w) + 12, y + 20, c2, A2) V.draw_search(x, y, mx, my, pressed, now, A, C)
local dd_new = false if V.ui_open and pressed and chip1_hit and not V.m_dd then pcall(V.cfg_export, false) V.m_saved_t = now end local sec = V.UI_TABS[V.m_tab] local lay = V.m_layout(sec)
local slide = (1 - V.ease((now - V.m_tab_t) / 0.22)) * 14 local gy = y + V.MTOP local cardy = gy + V.GH local names = V.TAB_GROUPS[sec.name] or { "MAIN", "OTHER" } for c = 1, _v(2) do
local colx = (cx0 + ((c - 1) * (V.MCOLW + 12))) + slide local rows = (c == 1) and sec.rows0 or sec.rows1 if rows > 0 then V.text(V.m_gh[c], colx + 4, gy + 4, names[c] or "", C.head)
V.rect(V.m_card[c], colx, cardy, V.MCOLW, (rows * V.ROWH) + (V.CPAD * 2), C.card, C.cardb) else V.hide(V.m_gh[c]); V.hide(V.m_card[c]) end end for i = 1, V.M_SLOTS do local s = V.m_slot[i] local w = sec.vis[i]
if i > sec.vn or not w then if s.kind then V.m_hide_slot(s); s.kind = nil end else if s.kind ~= w.t then V.m_hide_slot(s); s.kind = w.t end local L = lay[i] local colx = (cx0 + (L.col * (V.MCOLW + 12))) + slide
local ry = (cardy + V.CPAD) + L.y local rx = colx + 4 local rw = V.MCOLW - 8 local right = (colx + V.MCOLW) - 14 local hov = V.ui_open and not V.m_dd and not V.m_block and V.m_in(mx, my, rx, ry, rw, V.ROWH)
w.hk = V.m_lerp(w.hk or 0, hov and 1 or 0, ks) if hov then V.m_hover_w = w end if hov and w.t ~= "label" then V.m_hot = true end local fl = w.flash_t and (now - w.flash_t) < 1.8 if fl then
local fp = 0.5 + (0.5 * math.sin((now - w.flash_t) * 9)) V.rect(s.bg, rx, ry + 1, rw, V.ROWH - 2, V.mix(C.row, A, 0.18 + (0.22 * fp))) V.alpha(s.bg, 1) elseif w.hk > 0.03 and w.t ~= "label" then
V.rect(s.bg, rx, ry + 1, rw, V.ROWH - 2, C.row) V.alpha(s.bg, w.hk) else V.hide(s.bg) end local lcol = V.mix(C.text, V.C_WHITE, w.hk) if w.t == "label" then lcol = C.dim end V.text(s.lb, colx + 14, ry + 9, w.name, lcol)
if w.t == "toggle" then if hov and pressed then w:set(not w.v) end w.anim = V.m_lerp(w.anim or 0, w.v and 1 or 0, math.min(1, dt * 16)) local sx = right - 36 V.rect(s.a, sx, ry + 7, 36, 18, V.mix(C.off, A, w.anim))
V.rect(s.b, (sx + 3) + (18 * w.anim), ry + 10, 12, 12, V.C_WHITE) elseif w.t == "slider" then local vbw = 46 if not w.lw0 then w.lw0 = V.tw(w.name, 13) end local lw0 = w.lw0
local tx = math.max((colx + 14) + lw0 + 14, colx + math.floor(V.MCOLW * 0.42)) local tw0 = math.max(30, ((right - vbw) - 12) - tx)
if V.ui_open and pressed and not V.m_dd and V.m_in(mx, my, tx - 6, ry + 4, tw0 + 12, 24) then V.m_slider = w end if V.m_slider == w then if down then local f = math.max(0, math.min(1, (mx - tx) / tw0))
local nv = math.floor((w.min + ((w.max - w.min) * f)) + 0.5) if nv ~= w.v then w.v = nv end else V.m_slider = nil; w:set(w.v) end end local f = (w.v - w.min) / (w.max - w.min)
w.fa = V.m_lerp(w.fa or f, f, math.min(1, dt * 20)) V.rect(s.a, tx, ry + 14, tw0, 4, C.off) V.rect(s.b, tx, ry + 14, math.max(2, tw0 * w.fa), 4, A) V.rect(s.c, (tx + (tw0 * w.fa)) - 6, ry + 10, 12, 12, V.C_WHITE)
local vs = tostring(w.v) .. w.suf V.rect(s.p, right - vbw, ry + 6, vbw, 20, C.box, (V.m_slider == w) and A or C.box) if w.vs_c ~= vs then w.vs_c = vs w.vs_w = V.tw(vs, 12) end
V.text(s.v, ((right - vbw) + (vbw / 2)) - (w.vs_w / 2), ry + 9, vs, C.text) elseif w.t == "combo" then local bw = 128 local bx = right - bw local by = ry + 5 if hov and pressed and V.m_in(mx, my, bx, by, bw, 22) then
V.m_dd = w V.m_dd_t = now dd_new = true end V.rect(s.bx, bx, by, bw, 22, C.box, (V.m_dd == w) and A or C.box) local val = tostring(w.v) if string.len(val) > 15 then val = string.sub(val, 1, 14) .. ".." end
V.text(s.bt, bx + 10, by + 4, val, C.text) V.text(s.ar, (bx + bw) - 16, by + 4, (V.m_dd == w) and "^" or "v", C.dim) if V.m_dd == w then w._x = bx w._y = by + 26 w._w = bw end elseif w.t == "key" then local bw = 76
local bx = right - bw local cap = (V.capture_item == w) if hov and pressed and V.m_in(mx, my, bx, ry + 5, bw, 22) then V.capture_item = cap and nil or w end V.rect(s.bx, bx, ry + 5, bw, 22, C.box, cap and A or C.box)
local kt = cap and "..." or tostring(w.v) V.text(s.bt, (bx + (bw / 2)) - (V.tw(kt, 12) / 2), ry + 9, kt, cap and A or C.text) elseif w.t == "button" then V.text(s.ar, right - 8, ry + 8, ">", hov and A or C.dim)
if hov and pressed and w.cb then pcall(w.cb) end end end end if V.m_dd and V.m_dd._x then local w = V.m_dd local ih = 24 local cnt = math.min(w.on, V.M_DD) local ok = V.ease((now - V.m_dd_t) / 0.14)
local full = (cnt * ih) + 8 local hcur = math.max(6, full * ok) V.rect(V.m_ddshadow, w._x - 3, w._y - 1, w._w + 6, hcur + 6, C.shadow) V.alpha(V.m_ddshadow, 0.35) V.rect(V.m_ddbg, w._x, w._y, w._w, hcur, C.chip, C.cardb)
local any = false for i = 1, V.M_DD do local iy = (w._y + 4) + ((i - 1) * ih) if i <= cnt and (iy + ih) <= (w._y + hcur) then if V.m_in(mx, my, w._x, iy, w._w, ih) then any = true
V.rect(V.m_ddhl, w._x + 4, iy, w._w - 8, ih, V.mix(C.chip, A, 0.25)) if pressed and not dd_new then w:set(w.opts[i]); V.m_dd = nil end end
V.text(V.m_ddtx[i], w._x + 12, iy + 5, w.opts[i], (w.opts[i] == w.v) and A or C.text) else V.hide(V.m_ddtx[i]) end end if not any then V.hide(V.m_ddhl) end
if V.m_dd and pressed and not dd_new and not V.m_in(mx, my, w._x, w._y, w._w, full) then V.m_dd = nil end end if not V.m_dd then if not V.m_dd_hidden then V.m_dd_hidden = true
V.hide(V.m_ddshadow); V.hide(V.m_ddbg); V.hide(V.m_ddhl); for i = 1, V.M_DD do V.hide(V.m_ddtx[i]) end end else V.m_dd_hidden = false end V.draw_tip(x, y, k, A, C, ks) V.draw_profile(x, y, uy, k, now, A, C, ks)
V.hide(V.m_cursor) V.m_hot_prev = V.m_hot if V.ui_open and mx >= 0 then V.draw_dot_cursor(mx, my, A2, V.m_hot) else V.cur_hide() end end V.cfg_key = function(sec, w) return sec.name .. "/" .. (w.key or w.name) end
V.cfg_each = function(fn) for i = 1, V.UI_TN do local sec = V.UI_TABS[i] for j = 1, sec.n do local w = sec.items[j]
if not w.nosave and (w.t == "toggle" or w.t == "slider" or w.t == "combo" or w.t == "key") then fn(sec, w) end end end end V.cfg_each(function(_, w) w.def = w.v end) V.opt_has = function(opts, v)
for _, o in ipairs(opts) do if o == v then return true end end return false end V.cfg_set_status = function(msg) V.cfg_status.name = "status:  " .. msg end V.ui_relayout = function()
if V.vis_refresh then pcall(V.vis_refresh) end for i = 1, V.UI_TN do V.UI_TABS[i].lay = nil end end V.db_flush_ok = _v(nil) V.db_put = function(k, v) if not (database and database.write) then return false end
local data = v if json and json.stringify then local okj, sj = pcall(json.stringify, v) if okj and type(sj) == "string" then data = sj end end if not V.api("database.write", database.write, k, data) then return false end
if V.db_flush_ok ~= false and database.flush then pcall(database.flush) local okr, back = pcall(database.read, k) if okr and back ~= nil then V.db_flush_ok = true else V.db_flush_ok = false
V.api("database.write", database.write, k, data) end end return true end V.db_get = function(k) if not (database and database.read) then return nil end local ok, v = pcall(database.read, k)
if not ok or v == nil then return nil end if type(v) == "string" and json and json.parse then local okp, t = pcall(json.parse, v) if okp then return t end return nil end return v end V.slot_key = function(name)
return "neverpeek_cfg_" .. (string.gsub(string.lower(name or V.cfg_slot:get()), " ", "_")) end V.cfg_apply = function(t) local n = 0 V.cfg_each(function(sec, w) local v = t[V.cfg_key(sec, w)] if v == nil then return end
if w.t == "toggle" and type(v) == "boolean" then w:set(v); n = n + 1 elseif w.t == "slider" and type(v) == "number" then w:set(math.max(w.min, math.min(w.max, math.floor(v + 0.5)))); n = n + 1
elseif w.t == "combo" and type(v) == "string" and V.opt_has(w.opts, v) then w:set(v); n = n + 1 elseif w.t == "key" and type(v) == "string" and v ~= "" then w:set(v); n = n + 1 end end)
V.sa_refresh(); V.hf_refresh(); V.ui_relayout() return n end V.SV = {} V.SV_N = _v(0) V.cfg_each(function(_, w) V.SV_N = V.SV_N + 1; V.SV[V.SV_N] = w end) V.opt_index = function(opts, v)
for i, o in ipairs(opts) do if o == v then return i end end return nil end V.B36 = "0123456789abcdefghijklmnopqrstuvwxyz" V.h36 = function(str) local h = 5381
for k = 1, string.len(str) do h = ((h * 33) + string.byte(str, k)) % 1679616 end local o = "" for _ = 1, 4 do local d = h % 36 o = string.sub(V.B36, d + 1, d + 1) .. o h = math.floor(h / 36) end return o end V.K2W = {}
V.W2H = {} V.H2W = {} V.cfg_each(function(sec, w) local key = V.cfg_key(sec, w) V.K2W[key] = w local salt = "" local h = V.h36(key) while V.H2W[h] do salt = salt .. "#" h = V.h36(key .. salt) end V.H2W[h] = w
V.W2H[w] = h end)
V.NP2_B = "Anti-Aim/Enable AA|Anti-Aim/Yaw Modifier|Anti-Aim/Modifier Range|Anti-Aim/Yaw Offset|Anti-Aim/Desync Limit|Anti-Aim/Right Limit|Anti-Aim/Limit Jitter|Anti-Aim/Limit Jitter Speed|Anti-Aim/Inverter|Anti-Aim/Anti-Bruteforce|Anti-Aim/Anti-Brute Mode|Anti-Aim/Stage Reset|Anti-Aim/Modifier Hold|Anti-Aim/Randomize Hold|AA States/Enable AA States|AA States/Edit State|AA States/Standing override|AA States/Standing modifier|AA States/Standing range|AA States/Standing offset|AA States/Standing desync|AA States/Standing rdesync|AA States/Standing inverter|AA States/Standing defensive|AA States/Moving override|AA States/Moving modifier|AA States/Moving range|AA States/Moving offset|AA States/Moving desync|AA States/Moving rdesync|AA States/Moving inverter|AA States/Moving defensive|AA States/Crouching override|AA States/Crouching modifier|AA States/Crouching range|AA States/Crouching offset|AA States/Crouching desync|AA States/Crouching rdesync|AA States/Crouching inverter|AA States/Crouching defensive|AA States/In Air override|AA States/In Air modifier|AA States/In Air range|AA States/In Air offset|AA States/In Air desync|AA States/In Air rdesync|AA States/In Air inverter|AA States/In Air defensive|AA States/Air Crouch override|AA States/Air Crouch modifier|AA States/Air Crouch range|AA States/Air Crouch offset|AA States/Air Crouch desync|AA States/Air Crouch rdesync|AA States/Air Crouch inverter|AA States/Air Crouch defensive|Defensive/Defensive Flicks|Defensive/Flick Interval|Defensive/Flick Length|Defensive/Tickbase Shift|Defensive/Flick on Own Shot|Defensive/Break LC in Air|Defensive/Accurate Shooting|Defensive/Lag While Shooting|Defensive/AA While Shooting|Defensive/Spin Pitch On Shot|Defensive/Spin Duration|Defensive/Flick Trigger|Defensive/Peek Flick Window|Hard Flick/Enable Hard Flick|Hard Flick/Speed Mode|Hard Flick/Flick Speed|Hard Flick/Wave Fast Speed|Hard Flick/Wave Cycle|Hard Flick/Angle A|Hard Flick/Random A (0-86)|Hard Flick/Angle B|Hard Flick/Random B (0-86)|Resolver/Enable Lua Resolver|Resolver/Resolver Mode|Resolver/Built-in Mode Switch|Resolver/AA Classifier|Resolver/Per-Type Strategy|Resolver/Body First vs Defensive|Resolver/Disable Built-in|Resolver/Max Enemy Desync|Resolver/Jitter Detection|Resolver/Remember Hit Angle|Resolver/Angle Memory|Resolver/Damage-Validated Hits|Resolver/AA-Off Detection|Resolver/Anim Resolver|Resolver/BAim After Misses|Resolver/BAim Mode|Resolver/Remember Hard Targets|Resolver/Hard After Misses|Resolver/Safe Point After|Resolver/Enemy Pitch|Resolver/Log Shots To Console|Rage/Hitbox Priority|Rage/Wait For Scope|Rage/Scope Delay|Rage/Adaptive Hitchance|Rage/Base Hitchance|Rage/+ Per Spread Miss|Rage/Max Hitchance|Rage/Enable Fake Lag|Rage/Lag Limit|Rage/Lag Variability|Rage/Adaptive Lag|Rage/Force Safe Point|Rage/Lethal BAim|Rage/Smart DT|Rage/Dynamic Min Damage|Rage/Movement Hitchance|Rage/HC Walk|Rage/HC Run|Rage/HC Air|Visuals/Watermark|Visuals/Accent Color|Visuals/Font|Visuals/Stats Panel|Visuals/Keybinds List|Visuals/Crosshair Indicators|Visuals/Player State|Visuals/Hit Logs|Visuals/Drag Watermark|Visuals/Show Advanced|Visuals/Intro Animation|Visuals/Intro Dim + Vignette|Visuals/Performance|Visuals/Cull Crosshair In Scope|Visuals/Cull Max 3 Hit Logs|Visuals/Perf Profiler|Visuals/Tips Language|Visuals/Position X|Visuals/Position Y|Visuals/DT Bar|Visuals/Bullet Tracers|Visuals/Tracer Time|Visuals/DT Bar X|Visuals/DT Bar Y|Players/Players List|Players/Show Dead Players|Players/Highlight Target|Players/Miss Breakdown|Players/List X|Players/List Y|Extras/Killstreak Announcer|Extras/Announce Headshots|Extras/Multi-kill Window|Extras/Announcer Sound|Extras/Enemy Labels|Extras/Kill Say|Extras/Kill Say On|Extras/Kill Say Language|Extras/Kill Say Cooldown|Extras/Bees|Extras/Bee Style|Extras/Bee Count|Extras/Bee Orbit|Extras/Bee Size|Extras/Bees Swarm Kills|Extras/Pollen Trail|Extras/Music Player|Extras/Player Panel|Extras/Shuffle|Extras/Music Volume|Extras/Music Bass|Extras/Stop On Menu Open|Extras/Player X|Extras/Player Y|Grenades/Grenade Prediction|Grenades/Throw Strength|Grenades/Velocity Inheritance|Grenades/Draw Through Walls|Grenades/Zones Only On Ground|Grenades/Damage Labels|Grenades/Track Thrown Grenades|Grenades/Molotov Fire Zones|Grenades/Fire Lifetime|Grenades/Fire Radius|Grenades/HE Radius|Grenades/HE Damage|Binds/Anti-Aim Toggle|Binds/Pitch Up / Down|Binds/Manual Inverter|Binds/Menu Key|Binds/Music Next|Binds/Music Pause"
V.np2_order = function(skip) local L = {} local n = 0 for key in string.gmatch(V.NP2_B, "[^|]+") do if key ~= skip then n = n + 1 L[n] = V.K2W[key] or false end end return L end
V.NP2_ORD = { V.np2_order(""), V.np2_order("Visuals/Show Advanced") } V.cfg_enc = function(w) if w.t == "toggle" then return w.v and "1" or "0" elseif w.t == "slider" then return tostring(math.floor(w.v + 0.5))
elseif w.t == "combo" then return tostring(V.opt_index(w.opts, w.v) or "") elseif w.t == "key" then return (string.gsub(tostring(w.v), "[;:|%s]", "")) end return false end V.cfg_code = function() local body, n = "", 0
for i = 1, V.SV_N do local w = V.SV[i] if w.v ~= w.def then local e = V.cfg_enc(w) if e and e ~= "" then if n > 0 then body = body .. ";" end body = body .. V.W2H[w] .. ":" .. e n = n + 1 end end end
return "NP3|" .. body, n end V.CFG_NONE = {} V.cfg_valid = function(w, raw) if not w then return false end if w.t == "toggle" then return raw == "0" or raw == "1" end local x = tonumber(raw)
if w.t == "slider" then return x ~= nil and x >= w.min and x <= w.max end if w.t == "combo" then return x ~= nil and w.opts[x] ~= nil end if w.t == "key" then return x == nil end return false end
V.np2_score = function(L, parts) local sc = 0 for _, pr in ipairs(parts) do local w = L[pr[1]] if w == nil then sc = sc - 1 elseif w then if V.cfg_valid(w, pr[2]) then sc = sc + 1 else sc = sc - 1 end end end return sc
end V.code_apply = function(code) if type(code) ~= "string" then return nil end local ver, body = string.match(code, "NP([23])|([^%s]*)") if not ver then return nil end local vals = {} if ver == "3" then
for part in string.gmatch(body, "[^;]+") do local h, v = string.match(part, "^(%w+):(.+)$") if h and v and V.H2W[h] then vals[V.H2W[h]] = v end end else local parts = {} for part in string.gmatch(body, "[^;]+") do
local i, v = string.match(part, "^(%d+):(.+)$") local ii = tonumber(i) if ii and v then parts[#parts + 1] = { ii, v } end end local best = V.NP2_ORD[1]
if V.np2_score(V.NP2_ORD[2], parts) > V.np2_score(best, parts) then best = V.NP2_ORD[2] end for _, pr in ipairs(parts) do local w = best[pr[1]] if w then vals[w] = pr[2] end end end local n = 0 for i = 1, V.SV_N do
local w = V.SV[i] local raw = vals[w] local want = V.CFG_NONE if raw == nil then want = w.def elseif w.t == "toggle" then want = (raw == "1") elseif w.t == "slider" then local x = tonumber(raw)
if x then want = math.max(w.min, math.min(w.max, x)) end elseif w.t == "combo" then local x = tonumber(raw) if x then want = w.opts[x] end elseif w.t == "key" then want = raw end
if want ~= V.CFG_NONE and want ~= nil and want ~= w.v then w:set(want); n = n + 1 end end V.sa_refresh(); V.hf_refresh(); V.ui_relayout() return n end V.nat_box = _v(nil) V.nat_last = _v(nil) V.nat_tab = function()
local ok, t = pcall(menu.new_tab, "NeverPeek", "rbxassetid://10734975692") if ok and t then return t end return menu.new_tab("NeverPeek") end pcall(function() local nt = V.nat_tab()
local ns = nt:new_section("Config Code", "Left") V.nat_box = ns:new_textbox("NeverPeek code", "paste NeverPeek code here", "") end) V.INTRO = { id = "127655532136682", s = 3, e = 89 }
V.IM = { snd = false, state = "idle", t = -10, started = 0, tries = 0, fade_t = 0, trimmed = false, k = 0 } V.im_vol = function() return (V.mp_v:get() / 100) * 0.8 end V.im_kill = function() local M = V.IM if M.snd then
pcall(function() M.snd:Stop() end) pcall(function() M.snd:Destroy() end) end M.snd = false end V.im_parent_cam = function(snd) snd.Parent = workspace.CurrentCamera end V.im_make = function()
local snd = Instance.new("Sound") snd.SoundId = "rbxassetid://" .. V.INTRO.id snd.Volume = 0 snd.Looped = false snd.Name = "NP_IntroMusic" local eq = Instance.new("EqualizerSoundEffect") eq.LowGain = -6 eq.MidGain = 0
eq.HighGain = 0 eq.Parent = snd local okss, ss = pcall(function() return game:GetService("SoundService") end) if okss and ss then snd.Parent = ss else pcall(V.im_parent_cam, snd) end
pcall(V.set_prop, snd, "TimePosition", V.INTRO.s) snd:Play() return snd end V.im_info = function(snd) return { pos = tonumber(snd.TimePosition) or 0, loaded = snd.IsLoaded == true, alive = snd.Parent ~= nil } end
V.im_tick = function(now) local M = V.IM if M.state == "done" then return end if not V.mus_on:get() then V.im_kill() M.state = "done" return end if V.ui_hint_done and M.state ~= "fade" then if M.snd then M.state = "fade"
M.fade_t = now else M.state = "done" return end end if M.state == "fade" then local k = 1 - ((now - M.fade_t) / 1.2) if k <= 0 or not M.snd then V.im_kill() M.state = "done" else
pcall(V.set_prop, M.snd, "Volume", V.im_vol() * k * M.k) end return end if M.state == "idle" then if (now - M.t) < 1 then return end M.t = now if M.tries >= 3 then M.state = "done" return end M.tries = M.tries + 1
local ok, snd = pcall(V.im_make) if ok and snd then M.snd = snd M.state = "play" M.started = now M.trimmed = false M.k = 0 end return end if not M.snd then M.state = "idle" return end
local ok, inf = pcall(V.im_info, M.snd) local age = now - M.started if (not ok or not inf.alive or (not inf.loaded)) and age > 8 then V.im_kill() M.state = "idle" return end if not ok or not inf.loaded then return end
local a = V.INTRO.s local b = V.INTRO.e if not M.trimmed then if inf.pos < (a - 0.3) or inf.pos > b then pcall(V.set_prop, M.snd, "TimePosition", a) return end M.trimmed = true end if inf.pos >= b then
pcall(V.set_prop, M.snd, "TimePosition", a) M.trimmed = false M.k = 0 pcall(V.set_prop, M.snd, "Volume", 0) return end local k = math.min(1, math.max(0, (inf.pos - a) / 0.5), math.max(0, (b - inf.pos) / 1.5)) M.k = k
pcall(V.set_prop, M.snd, "Volume", V.im_vol() * k) end client.set_event_callback("paint", function() pcall(V.im_tick, V.now_s()) end) client.set_event_callback("shutdown", function() pcall(V.im_kill) end)
V.nat_get = function() if not V.nat_box then return nil end local ok, v = pcall(function() return V.nat_box:get() end) if ok and type(v) == "string" then return v end return nil end V.nat_set = function(v)
if not V.nat_box then return end pcall(function() V.nat_box:set(v) end) end V.cfg_export = function(quiet) local code, n = V.cfg_code() V.nat_last = code V.nat_set(code)
V.log(120, 200, 255, "[NP] CONFIG CODE (" .. tostring(n) .. " changes): " .. code) if not quiet then V.cfg_set_status("code printed to console  (" .. tostring(n) .. ")") end return code, n end
V.code_load = function(v, from) local n = V.code_apply(v) if n == nil then V.cfg_set_status("no NeverPeek code found") return false end V.cfg_set_status("code loaded  (" .. tostring(n) .. " changed)")
V.log(120, 200, 255, "[NP] config code loaded from " .. from .. " (" .. tostring(n) .. " changed)") return true end V.code_t = _v(0) client.set_event_callback("paint", function() local now = V.now_s()
if (now - V.code_t) < 1 then return end V.code_t = now local v = V.nat_get() if v and v ~= "" and v ~= V.nat_last then V.nat_last = v if string.find(v, "NP[23]|") then pcall(V.code_load, v, "cheat menu") end end end)
V.cfg_save = function() local t, n = {}, 0 V.cfg_each(function(sec, w) t[V.cfg_key(sec, w)] = w.v; n = n + 1 end) V.db_put(V.slot_key(), t) local _, ch = V.cfg_export(true)
V.cfg_set_status("saved " .. V.cfg_slot:get() .. " + code  (" .. tostring(ch) .. ")") end V.cfg_load = function(silent) local t = V.db_get(V.slot_key()) if type(t) ~= "table" then
V.cfg_set_status(V.cfg_slot:get() .. " is empty") return false end local n = V.cfg_apply(t) V.cfg_set_status("loaded " .. V.cfg_slot:get() .. "  (" .. tostring(n) .. ")")
if not silent then V.log(120, 200, 255, "[NP] config loaded: " .. V.cfg_slot:get()) end return true end V.cfg_reset = function() V.cfg_each(function(_, w) if w.def ~= nil then w:set(w.def) end end)
V.sa_refresh(); V.hf_refresh(); V.ui_relayout() V.cfg_set_status("reset to default") end V.preset_state = function(i, ov, mode, range, add, lim, inv, def) local c = V.SA[i]
c.ov:set(ov); c.mode:set(mode); c.range:set(range); c.add:set(add) c.lim:set(lim); c.rlim:set(lim); c.inv:set(inv); c.def:set(def) end V.preset_apply = function(name) V.aa_on:set(true) if name == "Jitter Meta" then
V.yaw_mode:set("Jitter"); V.yaw_range:set(42); V.yaw_add:set(0); V.desync_lim:set(86); V.desync_lim_r:set(86) V.inv_mode:set("Jitter"); V.anti_brute:set(true)
V.hf_on:set(false); V.def_on:set(true); V.def_interval:set(9); V.def_len:set(2) V.sa_on:set(true) V.preset_state(1, true, "Jitter", 38, 0, 86, "Jitter", true) V.preset_state(2, true, "Jitter", 52, 0, 86, "Jitter", true)
V.preset_state(3, true, "3-Way", 30, 0, 70, "Random", true) V.preset_state(4, true, "Random", 60, 0, 60, "Random", false) V.preset_state(5, true, "Random", 60, 0, 60, "Random", false) elseif name == "Static Desync" then
V.yaw_mode:set("Jitter"); V.yaw_range:set(0); V.yaw_add:set(0); V.desync_lim:set(86); V.desync_lim_r:set(86) V.inv_mode:set("Static"); V.anti_brute:set(true)
V.hf_on:set(false); V.def_on:set(true); V.def_interval:set(12); V.def_len:set(2) V.sa_on:set(false) elseif name == "Hard Flick" then V.yaw_mode:set("Jitter"); V.yaw_range:set(20); V.inv_mode:set("Jitter")
V.hf_on:set(true); V.hf_speed:set("Wave"); V.hf_ms:set(100); V.hf_ms2:set(30); V.hf_cycle:set(2) V.hf_a:set(86); V.hf_b:set(0); V.hf_rnd_a:set(false); V.hf_rnd_b:set(false) V.sa_on:set(false)
elseif name == "Defensive Heavy" then V.yaw_mode:set("Mixed"); V.yaw_range:set(42); V.inv_mode:set("Jitter") V.def_on:set(true); V.def_interval:set(6); V.def_len:set(3); V.def_shift:set(true)
V.def_onshot:set(true); V.def_air_lc:set(true); V.def_spin:set(true); V.def_spin_ms:set(350) V.hf_on:set(false); V.sa_on:set(false) end V.sa_refresh(); V.hf_refresh(); V.ui_relayout()
V.cfg_set_status("preset: " .. string.lower(name)) V.log(120, 200, 255, "[NP] AA preset applied: " .. name) end V.cfg_btn_s:set_callback(V.cfg_save) V.cfg_btn_l:set_callback(function() V.cfg_load(false) end)
V.cfg_btn_r:set_callback(V.cfg_reset) V.cfg_btn_p:set_callback(function() V.preset_apply(V.cfg_preset:get()) end) V.cfg_btn_e:set_callback(function() V.cfg_export(false) end) V.cfg_btn_i:set_callback(function()
local v = V.nat_get() if not v or v == "" then V.cfg_set_status("cheat menu code box is empty") return end V.nat_last = v V.code_load(v, "button") end) V.cfg_slot:set_callback(function(v)
local has = type(V.db_get(V.slot_key(v))) == "table" V.cfg_set_status(v .. (has and "  ·  saved" or "  ·  empty")) end) V.cfg_set_status(V.cfg_slot:get() .. "  ·  empty") V.KB_BINDS = {
{ item = V.k_aa, fn = function() V.aa_on:set(not V.aa_on:get()) end }, { item = V.k_pitch, fn = function() V.pitch_up = not V.pitch_up end }, { item = V.k_inv, fn = function() V.manual_inv = not V.manual_inv end },
{ item = V.k_menu, fn = function() V.ui_set_open(not V.ui_open) end }, } V.key_last = {} V.fire_key = function(k) if type(k) ~= "string" or k == "" or k == "Unknown" then return end local t = V.now_s()
if V.key_last[k] and (t - V.key_last[k]) < 0.08 then return end V.key_last[k] = t if input.is_typing and input.is_typing() then return end if V.ui_open and V.SR.focus and k ~= V.k_menu:get() then V.sr_key(k) return end
if V.ui_open then pcall(V.egg_key, k) end if V.capture_item then if k ~= "Escape" then V.capture_item:set(k) end V.capture_item = nil return end for _, kb in ipairs(V.KB_BINDS) do
if kb.item:get() == k and k ~= "None" then pcall(kb.fn) end end end V.KEYS_ALL = {} V.KEYS_N = _v(0) for i = 0, 25 do V.KEYS_N = V.KEYS_N + 1; V.KEYS_ALL[V.KEYS_N] = string.char(65 + i) end
for _, k in ipairs({ "Zero", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine", "F1", "F2", "F3", "F4", "F5", "F6", "F7", "F8", "F9", "F10", "F11", "F12", "LeftShift", "RightShift",
"LeftControl", "RightControl", "LeftAlt", "RightAlt", "Tab", "CapsLock", "Insert", "Home", "End", "Delete", "PageUp", "PageDown", "Backquote", "Escape", "Minus", "Equals", "LeftBracket", "RightBracket", "Semicolon",
"Quote", "Comma", "Period", "Slash", "BackSlash", "Up", "Down", "Left", "Right", "Backspace", "Space", "Return" }) do V.KEYS_N = V.KEYS_N + 1; V.KEYS_ALL[V.KEYS_N] = k end for i = 1, V.KEYS_N do local k = V.KEYS_ALL[i]
pcall(input.on_key_press, k, function() V.fire_key(k) end) end pcall(input.on_input_began, function(a) if type(a) == "string" then V.fire_key(a) return end local ok, kc = pcall(function() return a.KeyCode end)
if ok and kc ~= nil then local name = string.match(tostring(kc), "KeyCode%.(%w+)$") if name then V.fire_key(name) end end end) client.set_event_callback("shutdown", function()
if V.rs_conn then pcall(function() V.rs_conn:Disconnect() end) end pcall(input.set_cursor_unlocked, false) V.api("aa.reset", aa.reset)
if V.hc_applied or V.scope_blocked or V.baim_applied or V.sp_applied or (V.prio_applied and V.prio_applied ~= "Menu") then V.api("rage.reset", rage.reset) end pcall(plist.reset_all) V.PL = {} V.restore_builtin()
for i = 1, V.REG_N do local d = V.REG[i] pcall(function() d.Visible = false end) local okd = pcall(function() d:Destroy() end) if not okd then pcall(function() d:Remove() end) end pcall(function() d:Remove() end) end
_G.__rt_reset_stats = nil pcall(render.clear) end) V.log(180, 90, 255, "[NP] NeverPeek loaded | G - AA, F - pitch, X - invert") V.LIC = false pcall(function() if type(NP_LIC) == "table" then V.LIC = NP_LIC end end)
if V.LIC then local lic = "[NP] licensed to " .. tostring(V.LIC.name) .. " (key ..." .. tostring(V.LIC.key) .. ")" local okt, tnow = pcall(os.time) local exp = tonumber(V.LIC.exp) or 0 local soon = false
if exp > 0 and okt and tonumber(tnow) then local hours = math.max(0, (exp - tnow) / 3600) if V.LIC.kind == "sub" then lic = lic .. " | subscription, " .. string.format("%.0f", hours / 24) .. " days left"
soon = hours < (24 * 7) else lic = lic .. " | trial, " .. string.format("%.0f", hours) .. "h left" end elseif V.LIC.kind == "life" then lic = lic .. " | forever" end V.log(245, 184, 46, lic)
if soon then V.log(255, 120, 90, "[NP] your subscription ends soon, renew it to keep NeverPeek") end end do local okw, ws = pcall(function() return workspace end) local has_ws = okw and ws ~= nil
local has_ray = has_ws and pcall(function() return ws.Raycast end) local okr, rp = pcall(function() return RaycastParams end)
V.log(120, 200, 255, "[NP] ENV: workspace=" .. tostring(has_ws) .. " raycast=" .. tostring(has_ray) .. " RaycastParams=" .. tostring(okr and rp ~= nil)) end
