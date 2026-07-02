//@version=5
// ══════════════════════════════════════════════════════════════════════════════════════════════════ //
//# * ══════════════════════════════════════════════════════════════════════════════════════════════
//# *
//# * Study       : FibFans on Previous HTF HL
//# * Author      : © dgtrd, based on @faizannawaz1 idea
//# *
//# * Revision History
//# *  Release    : May 14, 2021
//# *  Update     : May 16, 2021 : Added all Pivot Points with Alerts
//# *  Update     : Mar 05, 2022 : handling user requests, slight enhancements and optimizations
//# *                               - added ability to switch fans to historical high/low levels
//# *                               - added ability to set alerts for fans levels
//# *
//# * ══════════════════════════════════════════════════════════════════════════════════════════════
// ══════════════════════════════════════════════════════════════════════════════════════════════════ //

// ---------------------------------------------------------------------------------------------- //
// Functions ------------------------------------------------------------------------------------ //


// ---------------------------------------------------------------------------------------------- //
// line/label/alert functions ------------------------------------------------------------------- //

f_drawLineX(_x1, _y1, _x2, _y2, _xloc, _extend, _color, _style, _width) =>
    var id = line.new(_x1, _y1, _x2, _y2, _xloc, _extend, _color, _style, _width)

    if _y1 > 0 and _y2 > 0
        line.set_xy1(id, _x1, _y1)
        line.set_xy2(id, _x2, _y2)
        line.set_color(id, _color)
    else
        line.set_xy1(id, _x1, close)
        line.set_xy2(id, _x2, close)
        line.set_color(id, #00000000)

f_drawLabelX(_x, _y, _text, _xloc, _yloc, _color, _style, _textcolor, _size, _textalign, _tooltip) =>
    var id = label.new(_x, _y, _text, _xloc, _yloc, _color, _style, _textcolor, _size, _textalign, _tooltip)
    label.set_text(id, _text)
    label.set_tooltip(id, _tooltip)
    
    if _y > 0
        label.set_xy(id, _x, _y)
        label.set_textcolor(id, _textcolor)
    else
        label.set_xy(id, _x, close)
        label.set_textcolor(id, #00000000)

f_crossingLevelX(_price, _level) =>
    _level > _price and _level < _price[1] or _level < _price and _level > _price[1]
    
f_getPrice(_x1, _y1, _x2, _y2) =>
    slope = (_y2 - _y1) / (_x2 - _x1)
    intercept = _y2 - slope * _x2
    price = slope * time + intercept

f_processPivotLevelX(_show, _x1, _y, _x2, _c, _s, _w, _lb, _tip) =>
    if _show
        style = _s == 'Solid' ? line.style_solid : _s == 'Dotted' ? line.style_dotted : line.style_dashed
        f_drawLineX(_x1, _y, _x2, _y, xloc.bar_time, extend.none, _c, style, _w)
        f_drawLabelX(_x2, _y, _lb, xloc.bar_time, yloc.price, #00000000, label.style_label_left, _c, size.normal, text.align_left, _tip + ' ' + _lb + '\n' + str.tostring(_y, format.mintick))

    if f_crossingLevelX(close, _y) and _show
        alert(syminfo.ticker + ' crossing ' + _tip + ' level ' + _lb + ' price ' + str.tostring(_y, format.mintick))

// ---------------------------------------------------------------------------------------------- //
// security() function free price calculations ---------------------------------------------------- //

f_htf_ohlc(_htf) =>
    var htf_o  = 0., var htf_h  = 0., var htf_l  = 0., htf_c  = close
    var htf_ox = 0., var htf_hx = 0., var htf_lx = 0., var htf_cx = 0.

    if ta.change(time(_htf))
        htf_ox := htf_o
        htf_o := open
        htf_hx := htf_h
        htf_h := high
        htf_lx := htf_l
        htf_l := low
        htf_cx := htf_c[1]
        htf_cx
    else
        htf_h := math.max(high, htf_h)
        htf_l := math.min(low, htf_l)
        htf_l

    [htf_ox, htf_hx, htf_lx, htf_cx, htf_o, htf_h, htf_l, htf_c]

// ---------------------------------------------------------------------------------------------- //
// pivot points function ------------------------------------------------------------------------ //

f_get_pivot(_o, _h, _l, _c, _o0, _type) =>
    var r6x = 0., var r5x = 0., var r4x = 0., var r3x = 0., var r2x = 0., var r1x = 0.
    var px  = 0.
    var s1x = 0., var s2x = 0., var s3x = 0., var s4x = 0., var s5x = 0., var s6x = 0.

    if _type == 'Camarilla'
        r5x := _h / _l * _c
        r4x := _c + (_h - _l) * 1.1 / 2
        r3x := _c + (_h - _l) * 1.1 / 4
        //r2x := _c + (_h - _l) * 1.1 / 6
        //r1x := _c + (_h - _l) * 1.1 / 12
        r6x := r5x + 1.168 * (r5x - r4x)
        //s1x := _c - (_h - _l) * 1.1 / 12
        //s2x := _c - (_h - _l) * 1.1 / 6
        s3x := _c - (_h - _l) * 1.1 / 4
        s4x := _c - (_h - _l) * 1.1 / 2
        s5x := _c - (r5x - _c)
        s6x := _c - (r6x - _c)
        s6x

    else if _type == 'Classic'
        px := math.avg(_h, _l, _c)
        s1x := px * 2 - _h
        s2x := px - (_h - _l)
        s3x := px - 2 * (_h - _l)
        s4x := px - 3 * (_h - _l)
        r1x := px * 2 - _l
        r2x := px + _h - _l
        r3x := px + 2 * (_h - _l)
        r4x := px + 3 * (_h - _l)
        r4x

    else if _type == 'DeMark'
        x = _c < _o ? _h + 2 * _l + _c : _c > _o ? 2 * _h + _l + _c : _h + _l + 2 * _c
        px := x / 4
        s1x := x / 2 - _h
        r1x := x / 2 - _l
        r1x

    else if _type == 'Traditional'
        px := math.avg(_h, _l, _c)
        s1x := px * 2 - _h
        s2x := px - (_h - _l)
        s3x := px * 2 - (2 * _h - _l)
        s4x := px * 3 - (3 * _h - _l)
        s5x := px * 4 - (4 * _h - _l)
        r1x := px * 2 - _l
        r2x := px + _h - _l
        r3x := px * 2 + _h - 2 * _l
        r4x := px * 3 + _h - 3 * _l
        r5x := px * 4 + _h - 4 * _l
        r5x

    else if _type == 'Fibonacci'
        px := math.avg(_h, _l, _c)
        r3x := px + _h - _l
        r2x := px + (_h - _l) * .618
        r1x := px + (_h - _l) * .382
        s1x := px - (_h - _l) * .382
        s2x := px - (_h - _l) * .618
        s3x := px - (_h - _l)
        s3x

    else if _type == 'Swing'
        r3x := _o0 + _h - _l
        r2x := _o0 + (_h - _l) * .618
        r1x := _o0 + (_h - _l) * .5
        s1x := _o0 - (_h - _l) * .5
        s2x := _o0 - (_h - _l) * .618
        s3x := _o0 - (_h - _l)
        s3x

    else if _type == 'Woodie'
        px := (_h + _l + 2 * _o0) / 4
        r1x := 2 * px - _l
        r2x := px + _h - _l
        r3x := _h + 2 * (px - _l)
        r4x := r3x + _h - _l
        s1x := 2 * px - _h
        s2x := px - (_h - _l)
        s3x := _l - 2 * (_h - px)
        s4x := s3x - (_h - _l)
        s4x

    [r6x, r5x, r4x, r3x, r2x, r1x, px, s1x, s2x, s3x, s4x, s5x, s6x]

// Functions  ----------------------------------------------------------------------------------- //
// ---------------------------------------------------------------------------------------------- //


indicator('FibFans on Previous HTF HL [FaizanNawaz] by DGT', 'FibFans on PHL ʙʏ DGT ☼☾', true, max_lines_count=200)

group_levels = 'Fibonacci Fan Settings'
tooltip_warn = 'in non 7/24 markets, fans will move automatically due to temporal gaps such as weekends, holidays etc'

htf_mode   = input.string('Auto', 'Time Frame', options=['Auto', 'User Defined'], inline='HTF', group=group_levels)
htf        = input.string('Weekly', '     or User Defined', options=['1 Hour', '4 Hour', 'Daily', 'Weekly', 'Monthly', 'Quarterly', 'Yearly'], group=group_levels)
htf_user   = htf == '1 Hour' ? '60' : htf == '4 Hour' ? '240' : htf == 'Daily' ? 'D' : htf == 'Weekly' ? 'W' : htf == 'Monthly' ? 'M' : htf == 'Quarterly' ? '3M' : '12M'
htf_auto   = timeframe.period == '1'   ? '60'  : 
             timeframe.period == '3'   ? '60'  : 
             timeframe.period == '5'   ? '240' : 
             timeframe.period == '15'  ? '240' : 
             timeframe.period == '30'  ? 'D'   : 
             timeframe.period == '45'  ? 'D'   : 
             timeframe.period == '60'  ? 'D'   : 
             timeframe.period == '120' ? 'W'   : 
             timeframe.period == '180' ? 'W'   : 
             timeframe.period == '240' ? 'W'   : 
             timeframe.period == 'D'   ? 'M'   : 
             timeframe.period == 'W'   ? '3M'  : '12M'
i_htf      = htf_mode == 'Auto' ? htf_auto : htf_user

i_isLeftFan  = input.bool(false, 'Disable Left Fans     |     Disable Right Fans', inline='FAN1', group=group_levels)
i_isRightFan = input.bool(false, '', inline='FAN1', group=group_levels)
i_prev       = input.bool(false, 'Move Left Fans Forward | Right Fans Backward', inline='FAN2', group=group_levels, tooltip=tooltip_warn)
i_back       = input.bool(false, '', inline='FAN2', group=group_levels)
histHL       = input.int (0, 'Switch Fans to Historical HTF High/Lows', minval = 0, group=group_levels)

i_isChangeHTF = input.bool(true, 'Higher TimeFrame Session Separator', group=group_levels)

tooltip_cpr = 'the central pivot range or cpr is price-based technical indicator, range can be used to predict the price movement\n' + 
               'the pivot points is a technical indicator that is used to determine the levels at which price may face support or resistance'

group_pick_a_pivot = 'Pivot Points Setup'
i_isCPR      = input.bool(true, 'Central Pivot Range | Pick a Pivot', inline='PVT', group=group_pick_a_pivot, tooltip=tooltip_cpr)
i_isPivot    = input.bool(true, '', inline='PVT', group=group_pick_a_pivot)
i_pickAPivot = input.string('Traditional', '', options=['Camarilla', 'Classic', 'DeMark', 'Fibonacci', 'Swing', 'Traditional', 'Woodie'], inline='PVT', group=group_pick_a_pivot)

i_dispPVT = input.bool(true, 'Subsequent Pivots, Hours Prior to Session End', inline='SPVT', group=group_pick_a_pivot)
i_when    = input.int(1, '', minval=1, inline='SPVT', group=group_pick_a_pivot)

i_show_r  = input.bool(true, 'Resistance Lines', inline='rLevel1', group=group_pick_a_pivot)
i_show_r1 = input.bool(true, 'R1', inline='rLevel1', group=group_pick_a_pivot)
i_show_r2 = input.bool(true, 'R2', inline='rLevel1', group=group_pick_a_pivot)
i_show_r3 = input.bool(true, 'R3', inline='rLevel1', group=group_pick_a_pivot)
i_show_r4 = input.bool(true, 'R4', inline='rLevel1', group=group_pick_a_pivot)
i_show_r5 = input.bool(true, 'R5', inline='rLevel1', group=group_pick_a_pivot)
i_show_r6 = input.bool(true, 'R6', inline='rLevel1', group=group_pick_a_pivot)
i_color_r = input.color(#e91e63, ' ', inline='rLevel', group=group_pick_a_pivot)
i_style_r = input.string('Solid', '', options=['Dashed', 'Dotted', 'Solid'], inline='rLevel', group=group_pick_a_pivot)
i_width_r = input.int(1, '', minval=1, inline='rLevel', group=group_pick_a_pivot)

i_show_p  = input.bool(true, 'Pivot Point Line', inline='ppLeve', group=group_pick_a_pivot)
i_color_p = input.color(#0000f0, ' ', inline='ppLeve', group=group_pick_a_pivot)
i_style_p = input.string('Solid', '', options=['Dashed', 'Dotted', 'Solid'], inline='ppLeve', group=group_pick_a_pivot)
i_width_p = input.int(1, '', minval=1, inline='ppLeve', group=group_pick_a_pivot)

i_show_s  = input.bool(true, 'Support Lines ', inline='sLevel1', group=group_pick_a_pivot)
i_show_s1 = input.bool(true, 'S1', inline='sLevel1', group=group_pick_a_pivot)
i_show_s2 = input.bool(true, 'S2', inline='sLevel1', group=group_pick_a_pivot)
i_show_s3 = input.bool(true, 'S3', inline='sLevel1', group=group_pick_a_pivot)
i_show_s4 = input.bool(true, 'S4', inline='sLevel1', group=group_pick_a_pivot)
i_show_s5 = input.bool(true, 'S5', inline='sLevel1', group=group_pick_a_pivot)
i_show_s6 = input.bool(true, 'S6', inline='sLevel1', group=group_pick_a_pivot)
i_color_s = input.color(#26a69a, ' ', inline='sLevel', group=group_pick_a_pivot)
i_style_s = input.string('Solid', '', options=['Dashed', 'Dotted', 'Solid'], inline='sLevel', group=group_pick_a_pivot)
i_width_s = input.int(1, '', minval=1, inline='sLevel', group=group_pick_a_pivot)

tooltip_range = 'graphical price range and price change display of the higher time frame (current and previous)\n\nhorizontally plotted htf candle'
i_isRange = input.bool(true, 'Price Range/Change Meter', group='Price Range/Change Meter', tooltip=tooltip_range)

// ---------------------------------------------------------------------------------------------- //
// Calculations --------------------------------------------------------------------------------- //

time_x12 = ta.valuewhen(ta.change(time(i_htf)), time, 2)
time_x11 = ta.valuewhen(ta.change(time(i_htf)), time, 1)
time_x1x = ta.valuewhen(ta.change(time(i_htf)), time, histHL + 1)
time_x1  = ta.valuewhen(ta.change(time(i_htf)), time, 0)
time_x2  = 2 * time_x1 - time_x11
time_x20 = 3 * time_x1 - 2 * time_x11

// ---------------------------------------------------------------------------------------------- //
// Previous Price Calculations ------------------------------------------------------------------ //

[O, H, L, C, O0, H0, L0, C0] = f_htf_ohlc(i_htf)
H := ta.valuewhen(ta.change(H), H, histHL)
L := ta.valuewhen(ta.change(L), L, histHL)
range_1 = 2 * (H - L)

if barstate.islast
    ohlcColor = O > C ? color.red : color.green

    f_drawLineX(time_x1x, H, i_isRightFan or i_back ? time_x2 : time_x20, H, xloc.bar_time, extend.none, ohlcColor, line.style_dashed, 1)
    f_drawLineX(time_x1 , H,                                     time_x2, H, xloc.bar_time, extend.none, ohlcColor, line.style_solid , 2)
    f_drawLabelX(i_isRightFan or i_back ? time_x2 : time_x20, H, 'PH', xloc.bar_time, yloc.price, #00000000, label.style_label_left , ohlcColor, size.normal, text.align_left, 'Previous HTF High\n' + str.tostring(H, format.mintick))
    f_drawLabelX(time_x1x                                   , H, 'PH', xloc.bar_time, yloc.price, #00000000, label.style_label_right, ohlcColor, size.normal, text.align_left, 'Previous HTF High\n' + str.tostring(H, format.mintick))

    f_drawLineX(time_x1x, L, i_isRightFan or i_back ? time_x2 : time_x20, L, xloc.bar_time, extend.none, ohlcColor, line.style_dashed, 1)
    f_drawLineX(time_x1 , L,                                     time_x2, L, xloc.bar_time, extend.none, ohlcColor, line.style_solid , 2)
    f_drawLabelX(i_isRightFan or i_back ? time_x2 : time_x20, L, 'PL', xloc.bar_time, yloc.price, #00000000, label.style_label_left , ohlcColor, size.normal, text.align_left, 'Previous HTF Low\n'  + str.tostring(L, format.mintick))
    f_drawLabelX(time_x1x                                   , L, 'PL', xloc.bar_time, yloc.price, #00000000, label.style_label_right, ohlcColor, size.normal, text.align_left, 'Previous HTF Low\n'  + str.tostring(L, format.mintick))

    if i_isChangeHTF
        f_drawLineX(time_x2, L, time_x2, H, xloc.bar_time, extend.both, color.new(color.blue, 89), line.style_solid, 3)
        if not i_isRightFan and not i_back
            f_drawLineX(time_x20, L, time_x20, H, xloc.bar_time, extend.both, color.new(color.blue, 89), line.style_solid, 3)

bgcolor(i_isChangeHTF and ta.change(time(i_htf)) ? color.new(color.blue, 89) : na)

if f_crossingLevelX(close, H)
    alert(syminfo.ticker + ' crossing previous high(' + str.tostring(histHL) + '), price - ' + str.tostring(H, format.mintick))
if f_crossingLevelX(close, L)
    alert(syminfo.ticker + ' crossing previous low('  + str.tostring(histHL) + '), price - ' + str.tostring(L, format.mintick)) 

// ---------------------------------------------------------------------------------------------- //
// Pivot Point Calculations --------------------------------------------------------------------- //

[R6, R5, R4, R3, R2, R1, P, S1, S2, S3, S4, S5, S6] = f_get_pivot(O , H , L , C , O0, i_pickAPivot)
[r6, r5, r4, r3, r2, r1, p, s1, s2, s3, s4, s5, s6] = f_get_pivot(O0, H0, L0, C0, O0, i_pickAPivot)
CPR  = math.avg(H , L , C ), BC  = math.avg(H , L ), TC  = 2 * CPR  - BC
CPR0 = math.avg(H0, L0, C0), BC0 = math.avg(H0, L0), TC0 = 2 * CPR0 - BC0

plot(i_isCPR ? CPR : na, 'Central Pivot Range' , ta.change(time(i_htf)) ? na : #4262ba)
plot(i_isCPR ? BC  : na, 'Bottom Central Level', ta.change(time(i_htf)) ? na : #fa8072)
plot(i_isCPR ? TC  : na, 'Top Central Level'   , ta.change(time(i_htf)) ? na : #9ef2e8)

when = barstate.islast and i_isCPR
f_processPivotLevelX(when, time_x1, CPR, time_x2, #4262ba, 'Solid', 1, 'CPR', 'Central Pivot Range' )
f_processPivotLevelX(when, time_x1, BC , time_x2, #fa8072, 'Solid', 1, 'BC' , 'Bottom Central Level')
f_processPivotLevelX(when, time_x1, TC , time_x2, #9ef2e8, 'Solid', 1, 'TC' , 'Top Central Level'   )

when := barstate.islast and i_isCPR and i_dispPVT and time_x2 - timenow < 3600000 * i_when
f_processPivotLevelX(when, time_x2, CPR0, time_x20, #4262ba, 'Solid', 1, 'CPR', 'Subsequent Central Pivot Range' )
f_processPivotLevelX(when, time_x2, BC0 , time_x20, #fa8072, 'Solid', 1, 'BC' , 'Subsequent Bottom Central Level')
f_processPivotLevelX(when, time_x2, TC0 , time_x20, #9ef2e8, 'Solid', 1, 'TC' , 'Subsequent Top Central Level'   )

when := barstate.islast and i_isPivot
f_processPivotLevelX(when and i_show_s and i_show_s1, time_x1, S1, time_x2, i_color_s, i_style_s, i_width_s, 'S1', i_pickAPivot)
f_processPivotLevelX(when and i_show_s and i_show_s2, time_x1, S2, time_x2, i_color_s, i_style_s, i_width_s, 'S2', i_pickAPivot)
f_processPivotLevelX(when and i_show_s and i_show_s3, time_x1, S3, time_x2, i_color_s, i_style_s, i_width_s, 'S3', i_pickAPivot)
f_processPivotLevelX(when and i_show_s and i_show_s4, time_x1, S4, time_x2, i_color_s, i_style_s, i_width_s, 'S4', i_pickAPivot)
f_processPivotLevelX(when and i_show_s and i_show_s5, time_x1, S5, time_x2, i_color_s, i_style_s, i_width_s, 'S5', i_pickAPivot)
f_processPivotLevelX(when and i_show_s and i_show_s6, time_x1, S6, time_x2, i_color_s, i_style_s, i_width_s, 'S6', i_pickAPivot)
f_processPivotLevelX(when and i_show_r and i_show_r1, time_x1, R1, time_x2, i_color_r, i_style_r, i_width_r, 'R1', i_pickAPivot)
f_processPivotLevelX(when and i_show_r and i_show_r2, time_x1, R2, time_x2, i_color_r, i_style_r, i_width_r, 'R2', i_pickAPivot)
f_processPivotLevelX(when and i_show_r and i_show_r3, time_x1, R3, time_x2, i_color_r, i_style_r, i_width_r, 'R3', i_pickAPivot)
f_processPivotLevelX(when and i_show_r and i_show_r4, time_x1, R4, time_x2, i_color_r, i_style_r, i_width_r, 'R4', i_pickAPivot)
f_processPivotLevelX(when and i_show_r and i_show_r5, time_x1, R5, time_x2, i_color_r, i_style_r, i_width_r, 'R5', i_pickAPivot)
f_processPivotLevelX(when and i_show_r and i_show_r6, time_x1, R6, time_x2, i_color_r, i_style_r, i_width_r, 'R6', i_pickAPivot)

when := barstate.islast and i_isPivot and CPR != P and P > 0
f_processPivotLevelX(when and i_show_p, time_x1, P, time_x2, i_color_p, i_style_p, i_width_p, 'P', i_pickAPivot)

when := barstate.islast and i_isPivot and i_dispPVT and time_x2 - timenow < 3600000 * i_when
f_processPivotLevelX(when and i_show_s and i_show_s1, time_x2, s1, time_x20, i_color_s, i_style_s, i_width_s, 'S1', 'Subsequent ' + i_pickAPivot)
f_processPivotLevelX(when and i_show_s and i_show_s2, time_x2, s2, time_x20, i_color_s, i_style_s, i_width_s, 'S2', 'Subsequent ' + i_pickAPivot)
f_processPivotLevelX(when and i_show_s and i_show_s3, time_x2, s3, time_x20, i_color_s, i_style_s, i_width_s, 'S3', 'Subsequent ' + i_pickAPivot)
f_processPivotLevelX(when and i_show_s and i_show_s4, time_x2, s4, time_x20, i_color_s, i_style_s, i_width_s, 'S4', 'Subsequent ' + i_pickAPivot)
f_processPivotLevelX(when and i_show_s and i_show_s5, time_x2, s5, time_x20, i_color_s, i_style_s, i_width_s, 'S5', 'Subsequent ' + i_pickAPivot)
f_processPivotLevelX(when and i_show_s and i_show_s6, time_x2, s6, time_x20, i_color_s, i_style_s, i_width_s, 'S6', 'Subsequent ' + i_pickAPivot)
f_processPivotLevelX(when and i_show_r and i_show_r1, time_x2, r1, time_x20, i_color_r, i_style_r, i_width_r, 'R1', 'Subsequent ' + i_pickAPivot)
f_processPivotLevelX(when and i_show_r and i_show_r2, time_x2, r2, time_x20, i_color_r, i_style_r, i_width_r, 'R2', 'Subsequent ' + i_pickAPivot)
f_processPivotLevelX(when and i_show_r and i_show_r3, time_x2, r3, time_x20, i_color_r, i_style_r, i_width_r, 'R3', 'Subsequent ' + i_pickAPivot)
f_processPivotLevelX(when and i_show_r and i_show_r4, time_x2, r4, time_x20, i_color_r, i_style_r, i_width_r, 'R4', 'Subsequent ' + i_pickAPivot)
f_processPivotLevelX(when and i_show_r and i_show_r5, time_x2, r5, time_x20, i_color_r, i_style_r, i_width_r, 'R5', 'Subsequent ' + i_pickAPivot)
f_processPivotLevelX(when and i_show_r and i_show_r6, time_x2, r6, time_x20, i_color_r, i_style_r, i_width_r, 'R6', 'Subsequent ' + i_pickAPivot)

when := barstate.islast and i_isPivot and i_dispPVT and CPR0 != p and p > 0 and time_x2 - timenow < 3600000 * i_when
f_processPivotLevelX(when and i_show_p, time_x2, P, time_x20, i_color_p, i_style_p, i_width_p, 'P', 'Subsequent ' + i_pickAPivot)

// ---------------------------------------------------------------------------------------------- //
// Fib Fans Calculations ------------------------------------------------------------------------ //

f_processFanLevels(_show, _level, _color, _width, _style, _line) =>
    if _show
        style = _style == 'Solid' ? line.style_solid : _style == 'Dotted' ? line.style_dotted : line.style_dashed

        if not i_isLeftFan
            f_drawLineX(i_prev ? time_x1 : time_x11, H, time_x2, H + range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)
            f_drawLineX(i_prev ? time_x1 : time_x11, H, time_x2, H - range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)
            f_drawLineX(i_prev ? time_x1 : time_x11, L, time_x2, L + range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)
            f_drawLineX(i_prev ? time_x1 : time_x11, L, time_x2, L - range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)

        if f_crossingLevelX(close, f_getPrice(i_prev ? time_x1 : time_x11, H, time_x2, H + range_1 * _level)) and not i_isLeftFan
            alert('AutoFibFan Left Upper: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_prev ? time_x1 : time_x11, H, time_x2, H + range_1 * _level), format.mintick))
        if f_crossingLevelX(close, f_getPrice(i_prev ? time_x1 : time_x11, H, time_x2, H - range_1 * _level)) and not i_isLeftFan
            alert('AutoFibFan Left Upper: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_prev ? time_x1 : time_x11, H, time_x2, H - range_1 * _level), format.mintick))
        if f_crossingLevelX(close, f_getPrice(i_prev ? time_x1 : time_x11, L, time_x2, L + range_1 * _level)) and not i_isLeftFan
            alert('AutoFibFan Left Lower: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_prev ? time_x1 : time_x11, L, time_x2, L + range_1 * _level), format.mintick))
        if f_crossingLevelX(close, f_getPrice(i_prev ? time_x1 : time_x11, L, time_x2, L - range_1 * _level)) and not i_isLeftFan
            alert('AutoFibFan Left Lower: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_prev ? time_x1 : time_x11, L, time_x2, L - range_1 * _level), format.mintick))
            
        if not i_isRightFan 
            f_drawLineX(i_back ? time_x2 : time_x20, H, time_x1, H + range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)
            f_drawLineX(i_back ? time_x2 : time_x20, H, time_x1, H - range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)
            f_drawLineX(i_back ? time_x2 : time_x20, L, time_x1, L + range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)
            f_drawLineX(i_back ? time_x2 : time_x20, L, time_x1, L - range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)

        if f_crossingLevelX(close, f_getPrice(i_back ? time_x2 : time_x20, H, time_x1, H + range_1 * _level)) and not i_isRightFan
            alert('AutoFibFan Right Upper: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_back ? time_x2 : time_x20, H, time_x1, H + range_1 * _level), format.mintick))
        if f_crossingLevelX(close, f_getPrice(i_back ? time_x2 : time_x20, H, time_x1, H - range_1 * _level)) and not i_isRightFan
            alert('AutoFibFan Right Upper: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_back ? time_x2 : time_x20, H, time_x1, H - range_1 * _level), format.mintick))
        if f_crossingLevelX(close, f_getPrice(i_back ? time_x2 : time_x20, L, time_x1, L + range_1 * _level)) and not i_isRightFan
            alert('AutoFibFan Right Lower: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_back ? time_x2 : time_x20, L, time_x1, L + range_1 * _level), format.mintick))
        if f_crossingLevelX(close, f_getPrice(i_back ? time_x2 : time_x20, L, time_x1, L - range_1 * _level)) and not i_isRightFan
            alert('AutoFibFan Right Lower: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_back ? time_x2 : time_x20, L, time_x1, L - range_1 * _level), format.mintick))

pshow_0_25   = input.bool(false, '', inline='pLevel_0_25', group=group_levels)
pvalue_0_25  = input.float(.25, '', step=.1, inline='pLevel_0_25', group=group_levels)
pcolor_0_25  = input.color(#f57c00, '', inline='pLevel_0_25', group=group_levels)
pwidth_0_25  = input.int(1, '', minval=1, inline='pLevel_0_25', group=group_levels)
pstyle_0_25  = input.string('Solid', '', options=['Dashed', 'Dotted', 'Solid'], inline='pLevel_0_25', group=group_levels)
f_processFanLevels(pshow_0_25, pvalue_0_25, pcolor_0_25, pwidth_0_25, pstyle_0_25, false)

pshow_0_382  = input.bool(false, '', inline='pLevel_0_382', group=group_levels)
pvalue_0_382 = input.float(.382, '', step=.1, inline='pLevel_0_382', group=group_levels)
pcolor_0_382 = input.color(#81c784, '', inline='pLevel_0_382', group=group_levels)
pwidth_0_382 = input.int(1, '', minval=1, inline='pLevel_0_382', group=group_levels)
pstyle_0_382 = input.string('Solid', '', options=['Dashed', 'Dotted', 'Solid'], inline='pLevel_0_382', group=group_levels)
f_processFanLevels(pshow_0_382, pvalue_0_382, pcolor_0_382, pwidth_0_382, pstyle_0_382, false)

pshow_0_50   = input.bool(false, '', inline='pLevel_0_50', group=group_levels)
pvalue_0_50  = input.float(.5, '', step=.1, inline='pLevel_0_50', group=group_levels)
pcolor_0_50  = input.color(#4caf50, '', inline='pLevel_0_50', group=group_levels)
pwidth_0_50  = input.int(1, '', minval=1, inline='pLevel_0_50', group=group_levels)
pstyle_0_50  = input.string('Solid', '', options=['Dashed', 'Dotted', 'Solid'], inline='pLevel_0_50', group=group_levels)
f_processFanLevels(pshow_0_50, pvalue_0_50, pcolor_0_50, pwidth_0_50, pstyle_0_50, false)

pshow_0_618  = input.bool(true, '', inline='pLevel_0_618', group=group_levels)
pvalue_0_618 = input.float(.618, '', step=.1, inline='pLevel_0_618', group=group_levels)
pcolor_0_618 = input.color(#009688, '', inline='pLevel_0_618', group=group_levels)
pwidth_0_618 = input.int(1, '', minval=1, inline='pLevel_0_618', group=group_levels)
pstyle_0_618 = input.string('Solid', '', options=['Dashed', 'Dotted', 'Solid'], inline='pLevel_0_618', group=group_levels)
f_processFanLevels(pshow_0_618, pvalue_0_618, pcolor_0_618, pwidth_0_618, pstyle_0_618, false)

pshow_0_75   = input.bool(false, '', inline='pLevel_0_75', group=group_levels)
pvalue_0_75  = input.float(.75, '', step=.1, inline='pLevel_0_75', group=group_levels)
pcolor_0_75  = input.color(#2196f3, '', inline='pLevel_0_75', group=group_levels)
pwidth_0_75  = input.int(1, '', minval=1, inline='pLevel_0_75', group=group_levels)
pstyle_0_75  = input.string('Solid', '', options=['Dashed', 'Dotted', 'Solid'], inline='pLevel_0_75', group=group_levels)
f_processFanLevels(pshow_0_75, pvalue_0_75, pcolor_0_75, pwidth_0_75, pstyle_0_75, false)

pshow_1      = input.bool(true, '', inline='pLevel_1', group=group_levels)
pvalue_1     = input.float(1., '', step=.1, inline='pLevel_1', group=group_levels)
pcolor_1     = input.color(#787b86, '', inline='pLevel_1', group=group_levels)
pwidth_1     = input.int(1, '', minval=1, inline='pLevel_1', group=group_levels)
pstyle_1     = input.string('Solid', '', options=['Dashed', 'Dotted', 'Solid'], inline='pLevel_1', group=group_levels)
f_processFanLevels(pshow_1, pvalue_1, pcolor_1, pwidth_1, pstyle_1, false)

pshow_1_618  = input.bool(true, '', inline='pLevel_1_618', group=group_levels)
pvalue_1_618 = input.float(1.618, '', step=.1, inline='pLevel_1_618', group=group_levels)
pcolor_1_618 = input.color(#2196f3, '', inline='pLevel_1_618', group=group_levels)
pwidth_1_618 = input.int(1, '', minval=1, inline='pLevel_1_618', group=group_levels)
pstyle_1_618 = input.string('Solid', '', options=['Dashed', 'Dotted', 'Solid'], inline='pLevel_1_618', group=group_levels)
f_processFanLevels(pshow_1_618, pvalue_1_618, pcolor_1_618, pwidth_1_618, pstyle_1_618, true)

pshow_2_618  = input.bool(false, '', inline='pLevel_2_618', group=group_levels)
pvalue_2_618 = input.float(2.618, '', step=.1, inline='pLevel_2_618', group=group_levels)
pcolor_2_618 = input.color(#f44336, '', inline='pLevel_2_618', group=group_levels)
pwidth_2_618 = input.int(1, '', minval=1, inline='pLevel_2_618', group=group_levels)
pstyle_2_618 = input.string('Solid', '', options=['Dashed', 'Dotted', 'Solid'], inline='pLevel_2_618', group=group_levels)
f_processFanLevels(pshow_2_618, pvalue_2_618, pcolor_2_618, pwidth_2_618, pstyle_2_618, true)

pshow_3_618  = input.bool(false, '', inline='pLevel_3_618', group=group_levels)
pvalue_3_618 = input.float(3.618, '', step=.1, inline='pLevel_3_618', group=group_levels)
pcolor_3_618 = input.color(#9c27b0, '', inline='pLevel_3_618', group=group_levels)
pwidth_3_618 = input.int(1, '', minval=1, inline='pLevel_3_618', group=group_levels)
pstyle_3_618 = input.string('Solid', '', options=['Dashed', 'Dotted', 'Solid'], inline='pLevel_3_618', group=group_levels)
f_processFanLevels(pshow_3_618, pvalue_3_618, pcolor_3_618, pwidth_3_618, pstyle_3_618, true)

// ══════════════════════════════════════════════════════════════════════════════════════════════════ //
//
// AddOns
// ══════════════════════════════════════════════════════════════════════════════════════════════════ //

// ---------------------------------------------------------------------------------------------- //
// -Inputs -------------------------------------------------------------------------------------- //
group_vol_vol = 'Volume Add-Ons'

tooltip_volume_spike_sign_of_exhaustion = 'Moments where\n' + 'huge volume detected : current volume is grater than the product of the theshold value and volume moving average'
tooltip_volume_weighted_colored_bars = 'Volume Weighted Colored Bars\nColors bars based on the bar\'s volume relative to volume moving average'

// ---------------------------------------------------------------------------------------------- //
// Volume Moving Average : Base ----------------------------------------------------------------- //

nzVolume = nz(volume)

i_vSMA = ta.sma(nzVolume, input.int(89, 'Volume Moving Average Length', group=group_vol_vol))

// ---------------------------------------------------------------------------------------------- //
// Volume Spike - Sign of Exhaustion ------------------------------------------------------------ //

i_vSpikeLb = input.bool(true, '🚦', inline='SRS1', group=group_vol_vol, tooltip=tooltip_volume_spike_sign_of_exhaustion)
i_vSpikeThresh = input.float(4.669, 'Volume Spike Theshold           ', minval=.1, step=.1, inline='SRS1', group=group_vol_vol)

exhaustVol = nzVolume > i_vSpikeThresh * i_vSMA
plotchar(i_vSpikeLb and nzVolume ? exhaustVol : na, 'Exhaustion Bar', '🚦', location.abovebar, size=size.tiny)
alertcondition(ta.crossover(nzVolume, i_vSMA * i_vSpikeThresh), 'Volume Spikes', 'sign of exhaustion, huge volume increase detected\n{{exchange}}:{{ticker}}->\nOpen = {{open}}, Current = {{close}},\nTime = {{time}}')

// ---------------------------------------------------------------------------------------------- //
// Volume Weighted Colored Bars ----------------------------------------------------------------- //

i_vwcb = input.bool(true, '', inline='VWC', group=group_vol_vol, tooltip=tooltip_volume_weighted_colored_bars)
i_vwcbHighThresh = input.float(1.618, 'VWCB :   High ', minval=1., step=.1, inline='VWC', group=group_vol_vol)
i_vwcbLowThresh = input.float(0.618, ' Low', minval=.1, step=.1, inline='VWC', group=group_vol_vol)

vwcbCol = nzVolume > i_vSMA * i_vwcbHighThresh ? close > open ? #006400 : #910000 : nzVolume < i_vSMA * i_vwcbLowThresh ? close < open ? #FF9800 : #7FFFD4 : na

barcolor(i_vwcb and nzVolume ? vwcbCol : na, title='Volume Weighted Colored Bars')

// Voloume / Volatility AddOns
// ══════════════════════════════════════════════════════════════════════════════════════════════════ //

// ══════════════════════════════════════════════════════════════════════════════════════════════════ //
// Price Range Meter AddOn

f_atr(_length) =>
    ta.atr(_length)
atr = request.security(syminfo.tickerid, i_htf, f_atr(14))

if i_isRange
    l = ta.valuewhen(ta.change(time(i_htf)), bar_index, 0) - ta.valuewhen(ta.change(time(i_htf)), bar_index, 1) 
    a = atr / 5
    t = time
    dif = time_x1 - time_x11
    tco = math.round(ta.change(t))
    f = math.min(L, L0)
    //------------------------------------------------------------------------------

    highVolatility = H0 - L0 > atr * 1.618

    if barstate.islast
        oo = math.round(l * (H0 - O0) / (H0 - L0))
        co = math.round(l * (H0 - close) / (H0 - L0))

        f_drawLineX(t - l  * tco - dif, f - a, t - dif           , f - a, xloc.bar_time, extend.none, color.gray                    , line.style_solid, 3)
        f_drawLineX(t - oo * tco - dif, f - a, t - co * tco - dif, f - a, xloc.bar_time, extend.none, O0 < close ? #006400 : #910000, line.style_solid, 7)

        f_drawLabelX(t - l * tco - dif , f - a, str.tostring(L0, format.mintick)   , xloc.bar_time, yloc.price, #00000000, label.style_label_right, O0 < close ? color.green : color.red, size.normal, text.align_center, 'HTF LOW' )
        f_drawLabelX(t - dif           , f - a, str.tostring(H0, format.mintick)   , xloc.bar_time, yloc.price, #00000000, label.style_label_left , O0 < close ? color.green : color.red, size.normal, text.align_center, 'HTF HIGH')
        f_drawLabelX(t - co * tco - dif, f - a, str.tostring(close, format.mintick), xloc.bar_time, yloc.price, O0 < close ? #006400 : #910000, label.style_label_up, color.white, size.normal, text.align_center, 'CURRENT PRICE')
        f_drawLabelX(t - math.round(l / 2) * tco - dif, f - a, 'PRICE RANGE - CHANGE ' + str.tostring(C0 - O0) + ' (' + str.tostring((C0 / C - 1) * 100, '#.##') + '%) (' + i_htf + ') ' + (highVolatility ? '⚡' : ''), xloc.bar_time, yloc.price, #00000000, label.style_label_down, O0 < close ? color.green : color.red, size.normal, text.align_center, (highVolatility ? '⚡ High Volatility Detected\nATR Value(14) ' + str.tostring(atr, '#.##') + '\n' : 'ATR Value(14) ' + str.tostring(atr, '#.##') + '\n') + 'Current Price Range ' + str.tostring((H0 - L0) , format.mintick))

        if not timeframe.ismonthly
            oo1 = math.round(l * (H - O) / (H - L))
            co1 = math.round(l * (H - C) / (H - L))

            f_drawLineX(t - l   * tco - dif, f - 3 * a, t - dif            , f - 3 * a, xloc.bar_time, extend.none, color.gray               , line.style_solid, 3)
            f_drawLineX(t - oo1 * tco - dif, f - 3 * a, t - co1 * tco - dif, f - 3 * a, xloc.bar_time, extend.none, O < C ? #006400 : #910000, line.style_solid, 7)

            f_drawLabelX(t - l * tco - dif, f - 3 * a, str.tostring(L, format.mintick), xloc.bar_time, yloc.price, #00000000, label.style_label_right, O < C ? color.green : color.red, size.normal, text.align_center, 'PREVIOUS HTF LOW' )
            f_drawLabelX(t - dif          , f - 3 * a, str.tostring(H,format.mintick) , xloc.bar_time, yloc.price, #00000000, label.style_label_left , O < C ? color.green : color.red, size.normal, text.align_center, 'PREVIOUS HTF HIGH')
            f_drawLabelX(t - math.round(l / 2) * tco - dif, f - 3 * a, 'PREVIOUS PRICE RANGE (' + i_htf + ')', xloc.bar_time, yloc.price, #00000000, label.style_label_up, O < C ? color.green : color.red, size.normal, text.align_center, 'PREVIOUS PRICE RANGE\nHIGH - LOW : ' + str.tostring((H - L), format.mintick))


var table logo = table.new(position.bottom_right, 1, 1)
if barstate.islast
    table.cell(logo, 0, 0, '☼☾  ', text_size=size.normal, text_color=color.teal)
