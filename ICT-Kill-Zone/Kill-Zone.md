// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © tradeforopp

//@version=5
indicator("ICT Killzones & Pivots [TFO]", "ICT Killzones & Pivots [TFO]", true, max_labels_count = 500, max_lines_count = 500, max_boxes_count = 500)

// ---------------------------------------- Constant Functions --------------------------------------------------
get_line_type(\_style) =>
result = switch \_style
'Solid' => line.style_solid
'Dotted' => line.style_dotted
'Dashed' => line.style_dashed
result

get_size(x) =>
result = switch x
'Auto' => size.auto
'Tiny' => size.tiny
'Small' => size.small
'Normal' => size.normal
'Large' => size.large
'Huge' => size.huge

get_table_pos(pos) =>
result = switch pos
"Bottom Center" => position.bottom_center
"Bottom Left" => position.bottom_left
"Bottom Right" => position.bottom_right
"Middle Center" => position.middle_center
"Middle Left" => position.middle_left
"Middle Right" => position.middle_right
"Top Center" => position.top_center
"Top Left" => position.top_left
"Top Right" => position.top_right
// ---------------------------------------- Constant Functions --------------------------------------------------

// ---------------------------------------- Inputs --------------------------------------------------
var g_SETTINGS = "Settings"
max_days = input.int(3, "Session Drawing Limit", 1, tooltip = "Only this many drawings will be kept on the chart, for each selected drawing type (killzone boxes, pivot lines, open lines, etc.)", group = g_SETTINGS)
tf_limit = input.timeframe("30", "Timeframe Limit", tooltip = "Drawings will not appear on timeframes greater than or equal to this", group = g_SETTINGS)
gmt_tz = input.string('America/New_York', "Timezone", options = ['America/New_York','GMT-12','GMT-11','GMT-10','GMT-9','GMT-8','GMT-7','GMT-6','GMT-5','GMT-4','GMT-3','GMT-2','GMT-1','GMT+0','GMT+1','GMT+2','GMT+3','GMT+4','GMT+5','GMT+6','GMT+7','GMT+8','GMT+9','GMT+10','GMT+11','GMT+12','GMT+13','GMT+14'], tooltip = "Note GMT is not adjusted to reflect Daylight Saving Time changes", group = g_SETTINGS)
lbl_size = get_size(input.string('Normal', "Label Size", options = ['Auto', 'Tiny', 'Small', 'Normal', 'Large', 'Huge'], tooltip = "The size of all labels", group = g_SETTINGS))
txt_color = input.color(color.black, "Text Color", tooltip = "The color of all label and table text", group = g_SETTINGS)
use_cutoff = input.bool(false, "Drawing Cutoff Time", inline = "CO", tooltip = "When enabled, all pivots and open price lines will stop extending at this time", group = g_SETTINGS)
cutoff = input.session("1800-1801", "", inline = "CO", group = g_SETTINGS)

var g_KZ = "Killzones"
show_kz = input.bool(true, "Show Killzone Boxes", inline = "KZ", group = g_KZ)
show_kz_text = input.bool(true, "Display Text", inline = "KZ", group = g_KZ)

use_asia = input.bool(true, "", inline = "ASIA", group = g_KZ)
as_txt = input.string("Asia", "", inline = "ASIA", group = g_KZ)
asia = input.session("2000-0000", "", inline = "ASIA", group = g_KZ)
as_color = input.color(color.blue, "", inline = "ASIA", group = g_KZ)

use_london = input.bool(true, "", inline = "LONDON", group = g_KZ)
lo_txt = input.string("London", "", inline = "LONDON", group = g_KZ)
london = input.session("0200-0500", "", inline = "LONDON", group = g_KZ)
lo_color = input.color(color.red, "", inline = "LONDON", group = g_KZ)

use_nyam = input.bool(true, "", inline = "NYAM", group = g_KZ)
na_txt = input.string("NY AM", "", inline = "NYAM", group = g_KZ)
nyam = input.session("0930-1100", "", inline = "NYAM", group = g_KZ)
na_color = input.color(#089981, "", inline = "NYAM", group = g_KZ)

use_nylu = input.bool(true, "", inline = "NYLU", group = g_KZ)
nl_txt = input.string("NY Lunch", "", inline = "NYLU", group = g_KZ)
nylu = input.session("1200-1300", "", inline = "NYLU", group = g_KZ)
nl_color = input.color(color.yellow, "", inline = "NYLU", group = g_KZ)

use_nypm = input.bool(true, "", inline = "NYPM", group = g_KZ)
np_txt = input.string("NY PM", "", inline = "NYPM", group = g_KZ)
nypm = input.session("1330-1600", "", inline = "NYPM", group = g_KZ)
np_color = input.color(color.purple, "", inline = "NYPM", group = g_KZ)

box_transparency = input.int(70, "Box Transparency", 0, 100, group = g_KZ)
text_transparency = input.int(50, "Text Transparency", 0, 100, group = g_KZ)

var g_LABELS = "Killzone Pivots"
show_pivots = input.bool(true, "Show Pivots", inline = "PV", group = g_LABELS)
use_alerts = input.bool(true, "Alert Broken Pivots", inline = "PV", tooltip = "The desired killzones must be enabled at the time that an alert is created, along with the show pivots option, in order for alerts to work", group = g_LABELS)
show_midpoints = input.bool(false, "Show Pivot Midpoints", inline = "mp", group = g_LABELS)
stop_midpoints = input.bool(true, "Stop Once Mitigated", inline = "mp", group = g_LABELS)
show_labels = input.bool(true, "Show Pivot Labels", inline = "LB", tooltip = "Show labels denoting each killzone's high and low. Optionally choose to show the price of each level. Right side will show labels on the right-hand side of the chart until they are reached", group = g_LABELS)
label_price = input.bool(false, "Display Price", inline = "LB", group = g_LABELS)
label_right = input.bool(false, "Right Side", inline = "LB", group = g_LABELS)
ext_pivots = input.string("Until Mitigated", "Extend Pivots...", options = ['Until Mitigated', 'Past Mitigation'], group = g_LABELS)
ext_which = input.string("Most Recent", "...From Which Sessions", options = ['Most Recent', 'All'], group = g_LABELS)

ash_str = input.string("AS.H", "Killzone 1 Labels", inline = "L_AS", group = g_LABELS)
asl_str = input.string("AS.L", "", inline = "L_AS", group = g_LABELS)

loh_str = input.string("LO.H", "Killzone 2 Labels", inline = "L_LO", group = g_LABELS)
lol_str = input.string("LO.L", "", inline = "L_LO", group = g_LABELS)

nah_str = input.string("NYAM.H", "Killzone 3 Labels", inline = "L_NA", group = g_LABELS)
nal_str = input.string("NYAM.L", "", inline = "L_NA", group = g_LABELS)

nlh_str = input.string("NYL.H", "Killzone 4 Labels", inline = "L_NL", group = g_LABELS)
nll_str = input.string("NYL.L", "", inline = "L_NL", group = g_LABELS)

nph_str = input.string("NYPM.H", "Killzone 5 Labels", inline = "L_NP", group = g_LABELS)
npl_str = input.string("NYPM.L", "", inline = "L_NP", group = g_LABELS)

kzp_style = get_line_type(input.string(defval = 'Solid', title = "Pivot Style", options = ['Solid', 'Dotted', 'Dashed'], inline = "KZP", group = g_LABELS))
kzp_width = input.int(1, "", inline = "KZP", group = g_LABELS)
kzm_style = get_line_type(input.string(defval = 'Dotted', title = "Midpoint Style", options = ['Solid', 'Dotted', 'Dashed'], inline = "KZM", group = g_LABELS))
kzm_width = input.int(1, "", inline = "KZM", group = g_LABELS)

var g_RNG = "Killzone Range"
show_range = input.bool(false, "Show Killzone Range", tooltip = "Show the most recent ranges of each selected killzone, from high to low", group = g_RNG)
show_range_avg = input.bool(true, "Show Average", tooltip = "Show the average range of each selected killzone", group = g_RNG)
range_avg = input.int(5, "Average Length", 0, tooltip = "This many previous sessions will be used to calculate the average. If there isn't enough data on the current chart, it will use as many sessions as possible", group = g_RNG)
range_pos = get_table_pos(input.string('Top Right', "Table Position", options = ['Bottom Center', 'Bottom Left', 'Bottom Right', 'Middle Center', 'Middle Left', 'Middle Right', 'Top Center', 'Top Left', 'Top Right'], group = g_RNG))
range_size = get_size(input.string('Normal', "Table Size", options = ['Auto', 'Tiny', 'Small', 'Normal', 'Large', 'Huge'], group = g_RNG))

var g_DWM = "Day - Week - Month"
sep_unlimited = input.bool(false, "Unlimited", tooltip = "Unlimited will show as many of the selected lines as possible. Otherwise, the session drawing limit will be used", group = g_DWM)
alert_HL = input.bool(false, "Alert High/Low Break", tooltip = "Alert when any selected highs and lows are traded through. The desired timeframe's high/low option must be enabled at the time that an alert is created", group = g_DWM)

show_d_open = input.bool(false, "D Open", inline = "DO", group = g_DWM)
dhl = input.bool(false, "High/Low", inline = "DO", tooltip = "", group = g_DWM)
ds = input.bool(false, "Separators", inline = "DO", tooltip = "Mark where a new day begins", group = g_DWM)
d_color = input.color(color.blue, "", inline = "DO", group = g_DWM)

show_w_open = input.bool(false, "W Open", inline = "WO", group = g_DWM)
whl = input.bool(false, "High/Low", inline = "WO", tooltip = "", group = g_DWM)
ws = input.bool(false, "Separators", inline = "WO", tooltip = "Mark where a new week begins", group = g_DWM)
w_color = input.color(#089981, "", inline = "WO", group = g_DWM)

show_m_open = input.bool(false, "M Open", inline = "MO", group = g_DWM)
mhl = input.bool(false, "High/Low", inline = "MO", tooltip = "", group = g_DWM)
ms = input.bool(false, "Separators", inline = "MO", tooltip = "Mark where a new month begins", group = g_DWM)
m_color = input.color(color.red, "", inline = "MO", group = g_DWM)

htf_style = get_line_type(input.string(defval = 'Solid', title = "Style", options = ['Solid', 'Dotted', 'Dashed'], inline = "D0", group = g_DWM))
htf_width = input.int(1, "", inline = "D0", group = g_DWM)

dow_labels = input.bool(true, "Day of Week Labels", inline = "DOW", group = g_DWM)
dow_yloc = input.string('Bottom', "", options = ['Top', 'Bottom'], inline = "DOW", group = g_DWM)
dow_xloc = input.string('Midnight', "", options = ['Midnight', 'Midday'], inline = "DOW", group = g_DWM)
dow_hide_wknd = input.bool(true, "Hide Weekend Labels", group = g_DWM)

var g_OPEN = "Opening Prices"
open_unlimited = input.bool(false, "Unlimited", tooltip = "Unlimited will show as many of the selected lines as possible. Otherwise, the session drawing limit will be used", group = g_OPEN)

use_h1 = input.bool(false, "", inline = "H1", group = g_OPEN)
h1_text = input.string("True Day Open", "", inline = "H1", group = g_OPEN)
h1 = input.session("0000-0001", "", inline = "H1", group = g_OPEN)
h1_color = input.color(color.black, "", inline = "H1", group = g_OPEN)

use_h2 = input.bool(false, "", inline = "H2", group = g_OPEN)
h2_text = input.string("06:00", "", inline = "H2", group = g_OPEN)
h2 = input.session("0600-0601", "", inline = "H2", group = g_OPEN)
h2_color = input.color(color.black, "", inline = "H2", group = g_OPEN)

use_h3 = input.bool(false, "", inline = "H3", group = g_OPEN)
h3_text = input.string("10:00", "", inline = "H3", group = g_OPEN)
h3 = input.session("1000-1001", "", inline = "H3", group = g_OPEN)
h3_color = input.color(color.black, "", inline = "H3", group = g_OPEN)

use_h4 = input.bool(false, "", inline = "H4", group = g_OPEN)
h4_text = input.string("14:00", "", inline = "H4", group = g_OPEN)
h4 = input.session("1400-1401", "", inline = "H4", group = g_OPEN)
h4_color = input.color(color.black, "", inline = "H4", group = g_OPEN)

use_h5 = input.bool(false, "", inline = "H5", group = g_OPEN)
h5_text = input.string("00:00", "", inline = "H5", group = g_OPEN)
h5 = input.session("0000-0001", "", inline = "H5", group = g_OPEN)
h5_color = input.color(color.black, "", inline = "H5", group = g_OPEN)

use_h6 = input.bool(false, "", inline = "H6", group = g_OPEN)
h6_text = input.string("00:00", "", inline = "H6", group = g_OPEN)
h6 = input.session("0000-0001", "", inline = "H6", group = g_OPEN)
h6_color = input.color(color.black, "", inline = "H6", group = g_OPEN)

use_h7 = input.bool(false, "", inline = "H7", group = g_OPEN)
h7_text = input.string("00:00", "", inline = "H7", group = g_OPEN)
h7 = input.session("0000-0001", "", inline = "H7", group = g_OPEN)
h7_color = input.color(color.black, "", inline = "H7", group = g_OPEN)

use_h8 = input.bool(false, "", inline = "H8", group = g_OPEN)
h8_text = input.string("00:00", "", inline = "H8", group = g_OPEN)
h8 = input.session("0000-0001", "", inline = "H8", group = g_OPEN)
h8_color = input.color(color.black, "", inline = "H8", group = g_OPEN)

hz_style = get_line_type(input.string(defval = 'Dotted', title = "Style", options = ['Solid', 'Dotted', 'Dashed'], inline = "H0", group = g_OPEN))
hz_width = input.int(1, "", inline = "H0", group = g_OPEN)

var g_VERTICAL = "Timestamps"
v_unlimited = input.bool(false, "Unlimited", tooltip = "Unlimited will show as many of the selected lines as possible. Otherwise, the session drawing limit will be used", group = g_VERTICAL)

use_v1 = input.bool(false, "", inline = "V1", group = g_VERTICAL)
v1 = input.session("0000-0001", "", inline = "V1", group = g_VERTICAL)
v1_color = input.color(color.black, "", inline = "V1", group = g_VERTICAL)

use_v2 = input.bool(false, "", inline = "V2", group = g_VERTICAL)
v2 = input.session("0800-0801", "", inline = "V2", group = g_VERTICAL)
v2_color = input.color(color.black, "", inline = "V2", group = g_VERTICAL)

use_v3 = input.bool(false, "", inline = "V3", group = g_VERTICAL)
v3 = input.session("1000-1001", "", inline = "V3", group = g_VERTICAL)
v3_color = input.color(color.black, "", inline = "V3", group = g_VERTICAL)

use_v4 = input.bool(false, "", inline = "V4", group = g_VERTICAL)
v4 = input.session("1200-1201", "", inline = "V4", group = g_VERTICAL)
v4_color = input.color(color.black, "", inline = "V4", group = g_VERTICAL)

vl_style = get_line_type(input.string(defval = 'Dotted', title = "Style", options = ['Solid', 'Dotted', 'Dashed'], inline = "V0", group = g_VERTICAL))
vl_width = input.int(1, "", inline = "V0", group = g_VERTICAL)
// ---------------------------------------- Inputs --------------------------------------------------

// ---------------------------------------- Variables & Constants --------------------------------------------------
type kz
string \_title

    box[] _box

    line[] _hi_line
    line[] _md_line
    line[] _lo_line

    label[] _hi_label
    label[] _lo_label

    bool[] _hi_valid
    bool[] _md_valid
    bool[] _lo_valid

    float[] _range_store
    float _range_current

type hz
line[] LN
label[] LB
bool[] CO

type dwm_hl
line[] hi_line
line[] lo_line
label[] hi_label
label[] lo_label
bool hit_high = false
bool hit_low = false

type dwm_info
string tf
float o = na
float h = na
float l = na
float ph = na
float pl = na

var as_kz = kz.new(as_txt, array.new_box(), array.new_line(), array.new_line(), array.new_line(), array.new_label(), array.new_label(), array.new_bool(), array.new_bool(), array.new_bool(), array.new_float())
var lo_kz = kz.new(lo_txt, array.new_box(), array.new_line(), array.new_line(), array.new_line(), array.new_label(), array.new_label(), array.new_bool(), array.new_bool(), array.new_bool(), array.new_float())
var na_kz = kz.new(na_txt, array.new_box(), array.new_line(), array.new_line(), array.new_line(), array.new_label(), array.new_label(), array.new_bool(), array.new_bool(), array.new_bool(), array.new_float())
var nl_kz = kz.new(nl_txt, array.new_box(), array.new_line(), array.new_line(), array.new_line(), array.new_label(), array.new_label(), array.new_bool(), array.new_bool(), array.new_bool(), array.new_float())
var np_kz = kz.new(np_txt, array.new_box(), array.new_line(), array.new_line(), array.new_line(), array.new_label(), array.new_label(), array.new_bool(), array.new_bool(), array.new_bool(), array.new_float())

var hz_1 = hz.new(array.new_line(), array.new_label(), array.new_bool())
var hz_2 = hz.new(array.new_line(), array.new_label(), array.new_bool())
var hz_3 = hz.new(array.new_line(), array.new_label(), array.new_bool())
var hz_4 = hz.new(array.new_line(), array.new_label(), array.new_bool())
var hz_5 = hz.new(array.new_line(), array.new_label(), array.new_bool())
var hz_6 = hz.new(array.new_line(), array.new_label(), array.new_bool())
var hz_7 = hz.new(array.new_line(), array.new_label(), array.new_bool())
var hz_8 = hz.new(array.new_line(), array.new_label(), array.new_bool())

var d_hl = dwm_hl.new(array.new_line(), array.new_line(), array.new_label(), array.new_label())
var w_hl = dwm_hl.new(array.new_line(), array.new_line(), array.new_label(), array.new_label())
var m_hl = dwm_hl.new(array.new_line(), array.new_line(), array.new_label(), array.new_label())

var d_info = dwm_info.new("D")
var w_info = dwm_info.new("W")
var m_info = dwm_info.new("M")

t_as = not na(time("", asia, gmt_tz))
t_lo = not na(time("", london, gmt_tz))
t_na = not na(time("", nyam, gmt_tz))
t_nl = not na(time("", nylu, gmt_tz))
t_np = not na(time("", nypm, gmt_tz))
t_co = not na(time("", cutoff, gmt_tz))

t_h1 = not na(time("", h1, gmt_tz))
t_h2 = not na(time("", h2, gmt_tz))
t_h3 = not na(time("", h3, gmt_tz))
t_h4 = not na(time("", h4, gmt_tz))
t_h5 = not na(time("", h5, gmt_tz))
t_h6 = not na(time("", h6, gmt_tz))
t_h7 = not na(time("", h7, gmt_tz))
t_h8 = not na(time("", h8, gmt_tz))

t_v1 = not na(time("", v1, gmt_tz))
t_v2 = not na(time("", v2, gmt_tz))
t_v3 = not na(time("", v3, gmt_tz))
t_v4 = not na(time("", v4, gmt_tz))

var d_sep_line = array.new_line()
var w_sep_line = array.new_line()
var m_sep_line = array.new_line()

var d_line = array.new_line()
var w_line = array.new_line()
var m_line = array.new_line()

var d_label = array.new_label()
var w_label = array.new_label()
var m_label = array.new_label()

var v1_line = array.new_line()
var v2_line = array.new_line()
var v3_line = array.new_line()
var v4_line = array.new_line()

var transparent = #ffffff00
var ext_current = ext_which == 'Most Recent'
var ext_past = ext_pivots == 'Past Mitigation'

update_dwm_info(dwm_info n) =>
if timeframe.change(n.tf)
n.ph := n.h
n.pl := n.l
n.o := open
n.h := high
n.l := low
else
n.h := math.max(high, n.h)
n.l := math.min(low, n.l)

if dhl or show_d_open
update_dwm_info(d_info)
if whl or show_w_open
update_dwm_info(w_info)
if mhl or show_m_open
update_dwm_info(m_info)
// ---------------------------------------- Variables & Constants --------------------------------------------------

// ---------------------------------------- Functions --------------------------------------------------
get_box_color(color c) =>
result = color.new(c, box_transparency)

get_text_color(color c) =>
result = color.new(c, text_transparency)
// ---------------------------------------- Functions --------------------------------------------------

// ---------------------------------------- Core Logic --------------------------------------------------
dwm_sep(string tf, bool use, line[] arr, color col) =>
if use
if timeframe.change(tf)
arr.unshift(line.new(bar_index, high\*1.0001, bar_index, low, style = htf_style, width = htf_width, extend = extend.both, color = col))
if not sep_unlimited and arr.size() > max_days
arr.pop().delete()

dwm_open(string tf, bool use, line[] lns, label[] lbls, dwm_info n, color col) =>
if use
if lns.size() > 0
lns.get(0).set_x2(time)
lbls.get(0).set_x(time)
if timeframe.change(tf)
lns.unshift(line.new(time, n.o, time, n.o, xloc = xloc.bar_time, style = htf_style, width = htf_width, color = col))
lbls.unshift(label.new(time, n.o, tf + " OPEN", xloc = xloc.bar_time, style = label.style_label_left, color = transparent, textcolor = txt_color, size = lbl_size))
if not sep_unlimited and lns.size() > max_days
lns.pop().delete()
lbls.pop().delete()

dwm_hl(string tf, bool use, dwm_hl hl, dwm_info n, color col) =>
if use
if hl.hi_line.size() > 0
hl.hi_line.get(0).set_x2(time)
hl.lo_line.get(0).set_x2(time)
hl.hi_label.get(0).set_x(time)
hl.lo_label.get(0).set_x(time)
if timeframe.change(tf)
hl.hi_line.unshift(line.new(time, n.ph, time, n.ph, xloc = xloc.bar_time, style = htf_style, width = htf_width, color = col))
hl.lo_line.unshift(line.new(time, n.pl, time, n.pl, xloc = xloc.bar_time, style = htf_style, width = htf_width, color = col))
hl.hi_label.unshift(label.new(time, n.ph, "P"+tf+"H", xloc = xloc.bar_time, style = label.style_label_left, color = transparent, textcolor = txt_color, size = lbl_size))
hl.lo_label.unshift(label.new(time, n.pl, "P"+tf+"L", xloc = xloc.bar_time, style = label.style_label_left, color = transparent, textcolor = txt_color, size = lbl_size))
hl.hit_high := false
hl.hit_low := false
if not sep_unlimited and hl.hi_line.size() > max_days
hl.hi_line.pop().delete()
hl.lo_line.pop().delete()
hl.hi_label.pop().delete()
hl.lo_label.pop().delete()
if hl.hi_line.size() > 0 and alert_HL
if not hl.hit_high and high > hl.hi_line.get(0).get_y1()
hl.hit_high := true
alert(str.format("Hit P{0}H", tf))
if not hl.hit_low and low < hl.lo_line.get(0).get_y1()
hl.hit_low := true
alert(str.format("Hit P{0}L", tf))

dwm() =>
if timeframe.in_seconds("") <= timeframe.in_seconds(tf_limit)
// DWM - Separators
dwm_sep("D", ds, d_sep_line, d_color)
dwm_sep("W", ws, w_sep_line, w_color)
dwm_sep("M", ms, m_sep_line, m_color)

        // DWM - Open Lines
        dwm_open("D", show_d_open, d_line, d_label, d_info, d_color)
        dwm_open("W", show_w_open, w_line, w_label, w_info, w_color)
        dwm_open("M", show_m_open, m_line, m_label, m_info, m_color)

        // DWM - Highs and Lows
        dwm_hl("D", dhl, d_hl, d_info, d_color)
        dwm_hl("W", whl, w_hl, w_info, w_color)
        dwm_hl("M", mhl, m_hl, m_info, m_color)

vline(bool use, bool t, line[] arr, color col) =>
if use
if t and not t[1]
arr.unshift(line.new(bar_index, high\*1.0001, bar_index, low, style = vl_style, width = vl_width, extend = extend.both, color = col))
if not v_unlimited
if arr.size() > max_days
arr.pop().delete()

vlines() =>
if timeframe.in_seconds("") <= timeframe.in_seconds(tf_limit)
vline(use_v1, t_v1, v1_line, v1_color)
vline(use_v2, t_v2, v2_line, v2_color)
vline(use_v3, t_v3, v3_line, v3_color)
vline(use_v4, t_v4, v4_line, v4_color)

hz_line(bool use, bool t, hz hz, string txt, color col) =>
if use
if t and not t[1]
hz.LN.unshift(line.new(bar_index, open, bar_index, open, style = hz_style, width = hz_width, color = col))
hz.LB.unshift(label.new(bar_index, open, txt, style = label.style_label_left, color = transparent, textcolor = txt_color, size = lbl_size))
array.unshift(hz.CO, false)
if not open_unlimited and hz.LN.size() > max_days
hz.LN.pop().delete()
hz.LB.pop().delete()
hz.CO.pop()
if not t and hz.CO.size() > 0
if not hz.CO.get(0)
hz.LN.get(0).set_x2(bar_index)
hz.LB.get(0).set_x(bar_index)
if (use_cutoff ? t_co : false)
hz.CO.set(0, true)

hz_lines() =>
if timeframe.in_seconds("") <= timeframe.in_seconds(tf_limit)
hz_line(use_h1, t_h1, hz_1, h1_text, h1_color)
hz_line(use_h2, t_h2, hz_2, h2_text, h2_color)
hz_line(use_h3, t_h3, hz_3, h3_text, h3_color)
hz_line(use_h4, t_h4, hz_4, h4_text, h4_color)
hz_line(use_h5, t_h5, hz_5, h5_text, h5_color)
hz_line(use_h6, t_h6, hz_6, h6_text, h6_color)
hz_line(use_h7, t_h7, hz_7, h7_text, h7_color)
hz_line(use_h8, t_h8, hz_8, h8_text, h8_color)

del_kz(kz k) =>
if k.\_box.size() > max_days
k.\_box.pop().delete()
if k.\_hi_line.size() > max_days
k.\_hi_line.pop().delete()
k.\_lo_line.pop().delete()
k.\_hi_valid.pop()
k.\_lo_valid.pop()
if show_midpoints
k.\_md_line.pop().delete()
k.\_md_valid.pop()
if k.\_hi_label.size() > max_days
k.\_hi_label.pop().delete()
k.\_lo_label.pop().delete()

update_price_string(label L, float P) =>
S = L.get_text()
pre = str.substring(S, 0, str.pos(S, " "))
str.trim(pre)
L.set_text(str.format("{0} ({1})", pre, P))

adjust_in_kz(kz kz, bool t) =>
if t
kz.\_box.get(0).set_right(time)
kz.\_box.get(0).set_top(math.max(kz.\_box.get(0).get_top(), high))
kz.\_box.get(0).set_bottom(math.min(kz.\_box.get(0).get_bottom(), low))

        kz._range_current := kz._box.get(0).get_top() - kz._box.get(0).get_bottom()

        if show_pivots and kz._hi_line.size() > 0
            kz._hi_line.get(0).set_x2(time)
            if high > kz._hi_line.get(0).get_y1()
                kz._hi_line.get(0).set_xy1(time, high)
                kz._hi_line.get(0).set_xy2(time, high)

            kz._lo_line.get(0).set_x2(time)
            if low < kz._lo_line.get(0).get_y1()
                kz._lo_line.get(0).set_xy1(time, low)
                kz._lo_line.get(0).set_xy2(time, low)

            if show_midpoints
                kz._md_line.get(0).set_x2(time)
                kz._md_line.get(0).set_xy1(time, math.avg(kz._hi_line.get(0).get_y2(), kz._lo_line.get(0).get_y2()))
                kz._md_line.get(0).set_xy2(time, math.avg(kz._hi_line.get(0).get_y2(), kz._lo_line.get(0).get_y2()))

        if show_labels and kz._hi_label.size() > 0
            if label_right
                kz._hi_label.get(0).set_x(time)
                kz._lo_label.get(0).set_x(time)
            if high > kz._hi_label.get(0).get_y()
                kz._hi_label.get(0).set_xy(time, high)
                if label_price
                    update_price_string(kz._hi_label.get(0), high)
            if low < kz._lo_label.get(0).get_y()
                kz._lo_label.get(0).set_xy(time, low)
                if label_price
                    update_price_string(kz._lo_label.get(0), low)

adjust_out_kz(kz kz, bool t) =>
if not t and kz.\_box.size() > 0
if t[1]
array.unshift(kz.\_range_store, kz.\_range_current)
if kz.\_range_store.size() > range_avg
kz.\_range_store.pop()

    if kz._box.size() > 0 and show_pivots
        for i = 0 to kz._box.size() - 1
            if not ext_current or (ext_current and i == 0)
                if ext_past ? true : (kz._hi_valid.get(i) == true)
                    kz._hi_line.get(i).set_x2(time)
                    if show_labels and label_right
                        kz._hi_label.get(i).set_x(time)
                if high > kz._hi_line.get(i).get_y1() and kz._hi_valid.get(i) == true
                    if use_alerts and i == 0
                        alert("Broke "+kz._title+" High", alert.freq_once_per_bar)
                    kz._hi_valid.set(i, false)
                    if show_labels and label_right
                        kz._hi_label.get(0).set_style(label.style_label_down)
                else if (use_cutoff ? t_co : false)
                    kz._hi_valid.set(i, false)

                if ext_past ? true : (kz._lo_valid.get(i) == true)
                    kz._lo_line.get(i).set_x2(time)
                    if show_labels and label_right
                        kz._lo_label.get(i).set_x(time)
                if low < kz._lo_line.get(i).get_y1() and kz._lo_valid.get(i) == true
                    if use_alerts and i == 0
                        alert("Broke "+kz._title+" Low", alert.freq_once_per_bar)
                    kz._lo_valid.set(i, false)
                    if show_labels and label_right
                        kz._lo_label.get(0).set_style(label.style_label_up)
                else if (use_cutoff ? t_co : false)
                    kz._lo_valid.set(i, false)

                if show_midpoints and not t
                    if stop_midpoints ? (kz._md_valid.get(i) == true) : true
                        kz._md_line.get(i).set_x2(time)
                        if kz._md_valid.get(i) == true and low <= kz._md_line.get(i).get_y1() and high >= kz._md_line.get(i).get_y1()
                            kz._md_valid.set(i, false)

            else
                break

manage_kz(kz kz, bool use, bool t, color c, string box_txt, string hi_txt, string lo_txt) =>
if timeframe.in_seconds("") <= timeframe.in_seconds(tf_limit) and use
if t and not t[1]
\_c = get_box_color(c)
\_t = get_text_color(c)
kz.\_box.unshift(box.new(time, high, time, low, xloc = xloc.bar_time, border_color = show_kz ? \_c : na, bgcolor = show_kz ? \_c : na, text = (show_kz and show_kz_text) ? box_txt : na, text_color = \_t))

            if show_pivots
                kz._hi_line.unshift(line.new(time, high, time, high, xloc = xloc.bar_time, style = kzp_style, color = c, width = kzp_width))
                kz._lo_line.unshift(line.new(time, low, time, low, xloc = xloc.bar_time, style = kzp_style, color = c, width = kzp_width))
                if show_midpoints
                    kz._md_line.unshift(line.new(time, math.avg(high, low), time, math.avg(high, low), xloc = xloc.bar_time, style = kzm_style, color = c, width = kzm_width))
                    array.unshift(kz._md_valid, true)

                array.unshift(kz._hi_valid, true)
                array.unshift(kz._lo_valid, true)

                if show_labels
                    _hi_txt = label_price ? str.format("{0} ({1})", hi_txt, high) : hi_txt
                    _lo_txt = label_price ? str.format("{0} ({1})", lo_txt, low)  : lo_txt
                    if label_right
                        kz._hi_label.unshift(label.new(time, high, _hi_txt, xloc = xloc.bar_time, color = transparent, textcolor = txt_color, style = label.style_label_left, size = lbl_size))
                        kz._lo_label.unshift(label.new(time, low,  _lo_txt, xloc = xloc.bar_time, color = transparent, textcolor = txt_color, style = label.style_label_left, size = lbl_size))
                    else
                        kz._hi_label.unshift(label.new(time, high, _hi_txt, xloc = xloc.bar_time, color = transparent, textcolor = txt_color, style = label.style_label_down, size = lbl_size))
                        kz._lo_label.unshift(label.new(time, low,  _lo_txt, xloc = xloc.bar_time, color = transparent, textcolor = txt_color, style = label.style_label_up, size = lbl_size))

            del_kz(kz)
        adjust_in_kz(kz, t)
        adjust_out_kz(kz, t)

manage_kz(as_kz, use_asia, t_as, as_color, as_txt, ash_str, asl_str)
manage_kz(lo_kz, use_london, t_lo, lo_color, lo_txt, loh_str, lol_str)
manage_kz(na_kz, use_nyam, t_na, na_color, na_txt, nah_str, nal_str)
manage_kz(nl_kz, use_nylu, t_nl, nl_color, nl_txt, nlh_str, nll_str)
manage_kz(np_kz, use_nypm, t_np, np_color, np_txt, nph_str, npl_str)

dwm()
vlines()
hz_lines()

new_dow_time = dow_xloc == 'Midday' ? time - timeframe.in_seconds("D") / 2 \* 1000 : time
new_day = dayofweek(new_dow_time, gmt_tz) != dayofweek(new_dow_time, gmt_tz)[1]

var dow_top = dow_yloc == 'Top'

var saturday = "SATURDAY"
var sunday = "SUNDAY"
var monday = "MONDAY"
var tuesday = "TUESDAY"
var wednesday = "WEDNESDAY"
var thursday = "THURSDAY"
var friday = "FRIDAY"

plotchar(dow_labels and timeframe.isintraday and dayofweek(new_dow_time, gmt_tz) == 1 and new_day and not dow_hide_wknd, location = dow_top ? location.top : location.bottom, char = "", textcolor = txt_color, text = sunday)
plotchar(dow_labels and timeframe.isintraday and dayofweek(new_dow_time, gmt_tz) == 2 and new_day, location = dow_top ? location.top : location.bottom, char = "", textcolor = txt_color, text = monday)
plotchar(dow_labels and timeframe.isintraday and dayofweek(new_dow_time, gmt_tz) == 3 and new_day, location = dow_top ? location.top : location.bottom, char = "", textcolor = txt_color, text = tuesday)
plotchar(dow_labels and timeframe.isintraday and dayofweek(new_dow_time, gmt_tz) == 4 and new_day, location = dow_top ? location.top : location.bottom, char = "", textcolor = txt_color, text = wednesday)
plotchar(dow_labels and timeframe.isintraday and dayofweek(new_dow_time, gmt_tz) == 5 and new_day, location = dow_top ? location.top : location.bottom, char = "", textcolor = txt_color, text = thursday)
plotchar(dow_labels and timeframe.isintraday and dayofweek(new_dow_time, gmt_tz) == 6 and new_day, location = dow_top ? location.top : location.bottom, char = "", textcolor = txt_color, text = friday)
plotchar(dow_labels and timeframe.isintraday and dayofweek(new_dow_time, gmt_tz) == 7 and new_day and not dow_hide_wknd, location = dow_top ? location.top : location.bottom, char = "", textcolor = txt_color, text = saturday)

get_min_days_stored() =>
store = array.new_int()
if as_kz.\_range_store.size() > 0
store.push(as_kz.\_range_store.size())
if lo_kz.\_range_store.size() > 0
store.push(lo_kz.\_range_store.size())
if na_kz.\_range_store.size() > 0
store.push(na_kz.\_range_store.size())
if nl_kz.\_range_store.size() > 0
store.push(nl_kz.\_range_store.size())
if np_kz.\_range_store.size() > 0
store.push(np_kz.\_range_store.size())
result = store.min()

set_table(table tbl, kz kz, int row, string txt, bool use, bool t, color col) =>
if use
table.cell(tbl, 0, row, txt, text_size = range_size, bgcolor = get_box_color(col), text_color = txt_color)
table.cell(tbl, 1, row, str.tostring(kz.\_range_current), text_size = range_size, bgcolor = t ? get_box_color(col) : na, text_color = txt_color)
if show_range_avg
table.cell(tbl, 2, row, str.tostring(kz.\_range_store.avg()), text_size = range_size, text_color = txt_color)

if show_range and barstate.islast
var tbl = table.new(range_pos, 10, 10, chart.bg_color, chart.fg_color, 2, chart.fg_color, 1)

    table.cell(tbl, 0, 0, "Killzone", text_size = range_size, text_color = txt_color)
    table.cell(tbl, 1, 0, "Range", text_size = range_size, text_color = txt_color)
    if show_range_avg
        table.cell(tbl, 2, 0, "Avg ("+str.tostring(get_min_days_stored())+")", text_size = range_size, text_color = txt_color)

    set_table(tbl, as_kz, 1, as_txt, use_asia, t_as, as_color)
    set_table(tbl, lo_kz, 2, lo_txt, use_london, t_lo, lo_color)
    set_table(tbl, na_kz, 3, na_txt, use_nyam, t_na, na_color)
    set_table(tbl, nl_kz, 4, nl_txt, use_nylu, t_nl, nl_color)
    set_table(tbl, np_kz, 5, np_txt, use_nypm, t_np, np_color)

// ---------------------------------------- Core Logic --------------------------------------------------
