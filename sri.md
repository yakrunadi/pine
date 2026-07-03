//@version=5
indicator('SMC + FibFans Pro Final MTF', 'SMC+FibFans Pro Final MTF', overlay = true, max_labels_count = 500, max_lines_count = 500, max_boxes_count = 500)

// Merged from workspace sources:
// 1) lux.md  -> Smart Money Concepts [LuxAlgo]
// 2) sri.md  -> FibFans on Previous HTF HL [FaizanNawaz] by DGT
// Notes:
// - Both modules remain in one overlay script.
// - Use the toggles below to disable one module if the chart becomes heavy.

MERGE_GROUP = '01. Master Controls'
enableSmartMoneyInput   = input.bool(true, 'Enable Smart Money Concepts (Lux)', group = MERGE_GROUP)
enableFibFansInput      = input.bool(true, 'Enable FibFans / Pivot / Volume (Sri)', group = MERGE_GROUP)
enableProOverlaysInput  = input.bool(true, 'Enable Pro Overlays (SMA / Volume Momentum / Ichimoku)', group = MERGE_GROUP)
performanceModeInput    = input.string('Normal', 'Performance Mode', options = ['Lite', 'Normal', 'Full'], group = MERGE_GROUP)
profilePresetInput      = input.string('Custom', 'Trading Preset', options = ['Custom', 'Scalping', 'Intraday', 'Swing'], group = MERGE_GROUP)
showProDashboardInput   = input.bool(true, 'Show Pro Dashboard', group = MERGE_GROUP)


// ===== MODULE 1: SMART MONEY CONCEPTS =====

// This work is licensed under a Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0) https://creativecommons.org/licenses/by-nc-sa/4.0/
// © LuxAlgo

//---------------------------------------------------------------------------------------------------------------------}
//CONSTANTS & STRINGS & INPUTS
//---------------------------------------------------------------------------------------------------------------------{
BULLISH_LEG                     = 1
BEARISH_LEG                     = 0

BULLISH                         = +1
BEARISH                         = -1

GREEN                           = #089981
RED                             = #F23645
BLUE                            = #2157f3
GRAY                            = #878b94
MONO_BULLISH                    = #b2b5be
MONO_BEARISH                    = #5d606b

HISTORICAL                      = 'Historical'
PRESENT                         = 'Present'

COLORED                         = 'Colored'
MONOCHROME                      = 'Monochrome'

ALL                             = 'All'
BOS                             = 'BOS'
CHOCH                           = 'CHoCH'

TINY                            = size.tiny
SMALL                           = size.small
NORMAL                          = size.normal

ATR                             = 'Atr'
RANGE                           = 'Cumulative Mean Range'

CLOSE                           = 'Close'
HIGHLOW                         = 'High/Low'

SOLID                           = '⎯⎯⎯'
DASHED                          = '----'
DOTTED                          = '····'

SMART_GROUP                     = '02. SMC • Core Settings'
INTERNAL_GROUP                  = '03. SMC • Internal Structure'
SWING_GROUP                     = '04. SMC • Swing Structure'
BLOCKS_GROUP                    = '05. SMC • Order Blocks'
EQUAL_GROUP                     = '06. SMC • Equal High / Low'
GAPS_GROUP                      = '07. SMC • Fair Value Gaps'
LEVELS_GROUP                    = '08. SMC • HTF Levels'
ZONES_GROUP                     = '09. SMC • Premium / Discount'

modeTooltip                     = 'Allows to display historical Structure or only the recent ones'
styleTooltip                    = 'Indicator color theme'
showTrendTooltip                = 'Display additional candles with a color reflecting the current trend detected by structure'
showInternalsTooltip            = 'Display internal market structure'
internalFilterConfluenceTooltip = 'Filter non significant internal structure breakouts'
showStructureTooltip            = 'Display swing market Structure'
showSwingsTooltip               = 'Display swing point as labels on the chart'
showHighLowSwingsTooltip        = 'Highlight most recent strong and weak high/low points on the chart'
showInternalOrderBlocksTooltip  = 'Display internal order blocks on the chart\n\nNumber of internal order blocks to display on the chart'
showSwingOrderBlocksTooltip     = 'Display swing order blocks on the chart\n\nNumber of internal swing blocks to display on the chart'
orderBlockFilterTooltip         = 'Method used to filter out volatile order blocks \n\nIt is recommended to use the cumulative mean range method when a low amount of data is available'
orderBlockMitigationTooltip     = 'Select what values to use for order block mitigation'
showEqualHighsLowsTooltip       = 'Display equal highs and equal lows on the chart'
equalHighsLowsLengthTooltip     = 'Number of bars used to confirm equal highs and equal lows'
equalHighsLowsThresholdTooltip  = 'Sensitivity threshold in a range (0, 1) used for the detection of equal highs & lows\n\nLower values will return fewer but more pertinent results'
showFairValueGapsTooltip        = 'Display fair values gaps on the chart'
fairValueGapsThresholdTooltip   = 'Filter out non significant fair value gaps'
fairValueGapsTimeframeTooltip   = 'Fair value gaps timeframe'
fairValueGapsExtendTooltip      = 'Determine how many bars to extend the Fair Value Gap boxes on chart'
showPremiumDiscountZonesTooltip = 'Display premium, discount, and equilibrium zones on chart'

modeInput                       = input.string( HISTORICAL, 'Mode',                     group = SMART_GROUP,    tooltip = modeTooltip, options = [HISTORICAL, PRESENT])
styleInput                      = input.string( COLORED,    'Style',                    group = SMART_GROUP,    tooltip = styleTooltip,options = [COLORED, MONOCHROME])
showTrendInput                  = input(        false,      'Color Candles',            group = SMART_GROUP,    tooltip = showTrendTooltip)

showInternalsInput              = input(        true,       'Show Internal Structure',  group = INTERNAL_GROUP, tooltip = showInternalsTooltip)
showInternalBullInput           = input.string( ALL,        'Bullish Structure',        group = INTERNAL_GROUP, inline = 'ibull', options = [ALL,BOS,CHOCH])
internalBullColorInput          = input(        GREEN,      '',                         group = INTERNAL_GROUP, inline = 'ibull')
showInternalBearInput           = input.string( ALL,        'Bearish Structure' ,       group = INTERNAL_GROUP, inline = 'ibear', options = [ALL,BOS,CHOCH])
internalBearColorInput          = input(        RED,        '',                         group = INTERNAL_GROUP, inline = 'ibear')
internalFilterConfluenceInput   = input(        false,      'Confluence Filter',        group = INTERNAL_GROUP, tooltip = internalFilterConfluenceTooltip)
internalStructureSize           = input.string( TINY,       'Internal Label Size',      group = INTERNAL_GROUP, options = [TINY,SMALL,NORMAL])

showStructureInput              = input(        true,       'Show Swing Structure',     group = SWING_GROUP,    tooltip = showStructureTooltip)
showSwingBullInput              = input.string( ALL,        'Bullish Structure',        group = SWING_GROUP,    inline = 'bull',    options = [ALL,BOS,CHOCH])
swingBullColorInput             = input(        GREEN,      '',                         group = SWING_GROUP,    inline = 'bull')
showSwingBearInput              = input.string( ALL,        'Bearish Structure',        group = SWING_GROUP,    inline = 'bear',    options = [ALL,BOS,CHOCH])
swingBearColorInput             = input(        RED,        '',                         group = SWING_GROUP,    inline = 'bear')
swingStructureSize              = input.string( SMALL,      'Swing Label Size',         group = SWING_GROUP,    options = [TINY,SMALL,NORMAL])
showSwingsInput                 = input(        false,      'Show Swings Points',       group = SWING_GROUP,    tooltip = showSwingsTooltip,inline = 'swings')
swingsLengthInput               = input.int(    50,         '',                         group = SWING_GROUP,    minval = 10,                inline = 'swings')
showHighLowSwingsInput          = input(        true,       'Show Strong/Weak High/Low',group = SWING_GROUP,    tooltip = showHighLowSwingsTooltip)

showInternalOrderBlocksInput    = input(        true,       'Internal Order Blocks' ,   group = BLOCKS_GROUP,   tooltip = showInternalOrderBlocksTooltip,   inline = 'iob')
internalOrderBlocksSizeInput    = input.int(    5,          '',                         group = BLOCKS_GROUP,   minval = 1, maxval = 20,                    inline = 'iob')
showSwingOrderBlocksInput       = input(        false,      'Swing Order Blocks',       group = BLOCKS_GROUP,   tooltip = showSwingOrderBlocksTooltip,      inline = 'ob')
swingOrderBlocksSizeInput       = input.int(    5,          '',                         group = BLOCKS_GROUP,   minval = 1, maxval = 20,                    inline = 'ob') 
orderBlockFilterInput           = input.string( 'Atr',      'Order Block Filter',       group = BLOCKS_GROUP,   tooltip = orderBlockFilterTooltip,          options = [ATR, RANGE])
orderBlockMitigationInput       = input.string( HIGHLOW,    'Order Block Mitigation',   group = BLOCKS_GROUP,   tooltip = orderBlockMitigationTooltip,      options = [CLOSE,HIGHLOW])
internalBullishOrderBlockColor  = input.color(color.new(#3179f5, 80), 'Internal Bullish OB',    group = BLOCKS_GROUP)
internalBearishOrderBlockColor  = input.color(color.new(#f77c80, 80), 'Internal Bearish OB',    group = BLOCKS_GROUP)
swingBullishOrderBlockColor     = input.color(color.new(#1848cc, 80), 'Bullish OB',             group = BLOCKS_GROUP)
swingBearishOrderBlockColor     = input.color(color.new(#b22833, 80), 'Bearish OB',             group = BLOCKS_GROUP)

showEqualHighsLowsInput         = input(        true,       'Equal High/Low',           group = EQUAL_GROUP,    tooltip = showEqualHighsLowsTooltip)
equalHighsLowsLengthInput       = input.int(    3,          'Bars Confirmation',        group = EQUAL_GROUP,    tooltip = equalHighsLowsLengthTooltip,      minval = 1)
equalHighsLowsThresholdInput    = input.float(  0.1,        'Threshold',                group = EQUAL_GROUP,    tooltip = equalHighsLowsThresholdTooltip,   minval = 0, maxval = 0.5, step = 0.1)
equalHighsLowsSizeInput         = input.string( TINY,       'Label Size',               group = EQUAL_GROUP,    options = [TINY,SMALL,NORMAL])

showFairValueGapsInput          = input(        false,      'Fair Value Gaps',          group = GAPS_GROUP,     tooltip = showFairValueGapsTooltip)
fairValueGapsThresholdInput     = input(        true,       'Auto Threshold',           group = GAPS_GROUP,     tooltip = fairValueGapsThresholdTooltip)
fairValueGapsTimeframeInput     = input.timeframe('',       'Timeframe',                group = GAPS_GROUP,     tooltip = fairValueGapsTimeframeTooltip)
fairValueGapsBullColorInput     = input.color(color.new(#00ff68, 70), 'Bullish FVG' , group = GAPS_GROUP)
fairValueGapsBearColorInput     = input.color(color.new(#ff0008, 70), 'Bearish FVG' , group = GAPS_GROUP)
fairValueGapsExtendInput        = input.int(    1,          'Extend FVG',               group = GAPS_GROUP,     tooltip = fairValueGapsExtendTooltip,       minval = 0)

showDailyLevelsInput            = input(        false,      'Daily',    group = LEVELS_GROUP,   inline = 'daily')
dailyLevelsStyleInput           = input.string( SOLID,      '',         group = LEVELS_GROUP,   inline = 'daily',   options = [SOLID,DASHED,DOTTED])
dailyLevelsColorInput           = input(        BLUE,       '',         group = LEVELS_GROUP,   inline = 'daily')
showWeeklyLevelsInput           = input(        false,      'Weekly',   group = LEVELS_GROUP,   inline = 'weekly')
weeklyLevelsStyleInput          = input.string( SOLID,      '',         group = LEVELS_GROUP,   inline = 'weekly',  options = [SOLID,DASHED,DOTTED])
weeklyLevelsColorInput          = input(        BLUE,       '',         group = LEVELS_GROUP,   inline = 'weekly')
showMonthlyLevelsInput          = input(        false,      'Monthly',   group = LEVELS_GROUP,   inline = 'monthly')
monthlyLevelsStyleInput         = input.string( SOLID,      '',         group = LEVELS_GROUP,   inline = 'monthly', options = [SOLID,DASHED,DOTTED])
monthlyLevelsColorInput         = input(        BLUE,       '',         group = LEVELS_GROUP,   inline = 'monthly')

showPremiumDiscountZonesInput   = input(        false,      'Premium/Discount Zones',   group = ZONES_GROUP , tooltip = showPremiumDiscountZonesTooltip)
premiumZoneColorInput           = input.color(  RED,        'Premium Zone',             group = ZONES_GROUP)
equilibriumZoneColorInput       = input.color(  GRAY,       'Equilibrium Zone',         group = ZONES_GROUP)
discountZoneColorInput          = input.color(  GREEN,      'Discount Zone',            group = ZONES_GROUP)

//---------------------------------------------------------------------------------------------------------------------}
//DATA STRUCTURES & VARIABLES
//---------------------------------------------------------------------------------------------------------------------{
// @type                            UDT representing alerts as bool fields
// @field internalBullishBOS        internal structure custom alert
// @field internalBearishBOS        internal structure custom alert
// @field internalBullishCHoCH      internal structure custom alert
// @field internalBearishCHoCH      internal structure custom alert
// @field swingBullishBOS           swing structure custom alert
// @field swingBearishBOS           swing structure custom alert
// @field swingBullishCHoCH         swing structure custom alert
// @field swingBearishCHoCH         swing structure custom alert
// @field internalBullishOrderBlock internal order block custom alert
// @field internalBearishOrderBlock internal order block custom alert
// @field swingBullishOrderBlock    swing order block custom alert
// @field swingBearishOrderBlock    swing order block custom alert
// @field equalHighs                equal high low custom alert
// @field equalLows                 equal high low custom alert
// @field bullishFairValueGap       fair value gap custom alert
// @field bearishFairValueGap       fair value gap custom alert
type alerts
    bool internalBullishBOS         = false
    bool internalBearishBOS         = false
    bool internalBullishCHoCH       = false
    bool internalBearishCHoCH       = false
    bool swingBullishBOS            = false
    bool swingBearishBOS            = false
    bool swingBullishCHoCH          = false
    bool swingBearishCHoCH          = false
    bool internalBullishOrderBlock  = false
    bool internalBearishOrderBlock  = false
    bool swingBullishOrderBlock     = false
    bool swingBearishOrderBlock     = false
    bool equalHighs                 = false
    bool equalLows                  = false
    bool bullishFairValueGap        = false
    bool bearishFairValueGap        = false

// @type                            UDT representing last swing extremes (top & bottom)
// @field top                       last top swing price
// @field bottom                    last bottom swing price
// @field barTime                   last swing bar time
// @field barIndex                  last swing bar index
// @field lastTopTime               last top swing time
// @field lastBottomTime            last bottom swing time
type trailingExtremes
    float top
    float bottom
    int barTime
    int barIndex
    int lastTopTime
    int lastBottomTime

// @type                            UDT representing Fair Value Gaps
// @field top                       top price
// @field bottom                    bottom price
// @field bias                      bias (BULLISH or BEARISH)
// @field topBox                    top box
// @field bottomBox                 bottom box
type fairValueGap
    float top
    float bottom
    int bias
    box topBox
    box bottomBox

// @type                            UDT representing trend bias
// @field bias                      BULLISH or BEARISH
type trend
    int bias    

// @type                            UDT representing Equal Highs Lows display
// @field l_ine                     displayed line
// @field l_abel                    displayed label
type equalDisplay
    line l_ine      = na
    label l_abel    = na

// @type                            UDT representing a pivot point (swing point) 
// @field currentLevel              current price level
// @field lastLevel                 last price level
// @field crossed                   true if price level is crossed
// @field barTime                   bar time
// @field barIndex                  bar index    
type pivot
    float currentLevel
    float lastLevel
    bool crossed
    int barTime     = time
    int barIndex    = bar_index

// @type                            UDT representing an order block
// @field barHigh                   bar high
// @field barLow                    bar low
// @field barTime                   bar time
// @field bias                      BULLISH or BEARISH
type orderBlock
    float barHigh
    float barLow
    int barTime    
    int bias

// @variable                        current swing pivot high    
var pivot swingHigh                 = pivot.new(na,na,false)
// @variable                        current swing pivot low
var pivot swingLow                  = pivot.new(na,na,false)
// @variable                        current internal pivot high
var pivot internalHigh              = pivot.new(na,na,false)
// @variable                        current internal pivot low
var pivot internalLow               = pivot.new(na,na,false)
// @variable                        current equal high pivot
var pivot equalHigh                 = pivot.new(na,na,false)
// @variable                        current equal low pivot
var pivot equalLow                  = pivot.new(na,na,false)
// @variable                        swing trend bias
var trend swingTrend                = trend.new(0)
// @variable                        internal trend bias
var trend internalTrend             = trend.new(0)
// @variable                        equal high display
var equalDisplay equalHighDisplay   = equalDisplay.new()
// @variable                        equal low display
var equalDisplay equalLowDisplay    = equalDisplay.new()
// @variable                        storage for fairValueGap UDTs
var array<fairValueGap> fairValueGaps = array.new<fairValueGap>()
// @variable                        storage for parsed highs
var array<float> parsedHighs        = array.new<float>()
// @variable                        storage for parsed lows
var array<float> parsedLows         = array.new<float>()
// @variable                        storage for raw highs
var array<float> highs              = array.new<float>()
// @variable                        storage for raw lows
var array<float> lows               = array.new<float>()
// @variable                        storage for bar time values
var array<int> times                = array.new<int>()
// @variable                        last trailing swing high and low
var trailingExtremes trailing       = trailingExtremes.new()
// @variable                                storage for orderBlock UDTs (swing order blocks)
var array<orderBlock> swingOrderBlocks      = array.new<orderBlock>()
// @variable                                storage for orderBlock UDTs (internal order blocks)
var array<orderBlock> internalOrderBlocks   = array.new<orderBlock>()
// @variable                                storage for swing order blocks boxes
var array<box> swingOrderBlocksBoxes        = array.new<box>()
// @variable                                storage for internal order blocks boxes
var array<box> internalOrderBlocksBoxes     = array.new<box>()
// @variable                        color for swing bullish structures
var swingBullishColor               = styleInput == MONOCHROME ? MONO_BULLISH : swingBullColorInput
// @variable                        color for swing bearish structures
var swingBearishColor               = styleInput == MONOCHROME ? MONO_BEARISH : swingBearColorInput
// @variable                        color for bullish fair value gaps
var fairValueGapBullishColor        = styleInput == MONOCHROME ? color.new(MONO_BULLISH,70) : fairValueGapsBullColorInput
// @variable                        color for bearish fair value gaps
var fairValueGapBearishColor        = styleInput == MONOCHROME ? color.new(MONO_BEARISH,70) : fairValueGapsBearColorInput
// @variable                        color for premium zone
var premiumZoneColor                = styleInput == MONOCHROME ? MONO_BEARISH : premiumZoneColorInput
// @variable                        color for discount zone
var discountZoneColor               = styleInput == MONOCHROME ? MONO_BULLISH : discountZoneColorInput 
// @variable                        bar index on current script iteration
varip int currentBarIndex           = bar_index
// @variable                        bar index on last script iteration
varip int lastBarIndex              = bar_index
// @variable                        alerts in current bar
alerts currentAlerts                = alerts.new()
// @variable                        time at start of chart
var initialTime                     = time

// we create the needed boxes for displaying order blocks at the first execution
if barstate.isfirst
    if showSwingOrderBlocksInput
        for index = 1 to swingOrderBlocksSizeInput
            swingOrderBlocksBoxes.push(box.new(na,na,na,na,xloc = xloc.bar_time,extend = extend.right))
    if showInternalOrderBlocksInput
        for index = 1 to internalOrderBlocksSizeInput
            internalOrderBlocksBoxes.push(box.new(na,na,na,na,xloc = xloc.bar_time,extend = extend.right))

// @variable                        source to use in bearish order blocks mitigation
bearishOrderBlockMitigationSource   = orderBlockMitigationInput == CLOSE ? close : high
// @variable                        source to use in bullish order blocks mitigation
bullishOrderBlockMitigationSource   = orderBlockMitigationInput == CLOSE ? close : low
// @variable                        default volatility measure
atrMeasure                          = ta.atr(200)
// @variable                        parsed volatility measure by user settings
volatilityMeasure                   = orderBlockFilterInput == ATR ? atrMeasure : ta.cum(ta.tr)/bar_index
// @variable                        true if current bar is a high volatility bar
highVolatilityBar                   = (high - low) >= (2 * volatilityMeasure)
// @variable                        parsed high
parsedHigh                          = highVolatilityBar ? low : high
// @variable                        parsed low
parsedLow                           = highVolatilityBar ? high : low

// we store current values into the arrays at each bar
parsedHighs.push(parsedHigh)
parsedLows.push(parsedLow)
highs.push(high)
lows.push(low)
times.push(time)

//---------------------------------------------------------------------------------------------------------------------}
//USER-DEFINED FUNCTIONS
//---------------------------------------------------------------------------------------------------------------------{
// @function            Get the value of the current leg, it can be 0 (bearish) or 1 (bullish)
// @returns             int
leg(int size) =>
    var leg     = 0    
    newLegHigh  = high[size] > ta.highest( size)
    newLegLow   = low[size]  < ta.lowest(  size)
    
    if newLegHigh
        leg := BEARISH_LEG
    else if newLegLow
        leg := BULLISH_LEG
    leg

// @function            Identify whether the current value is the start of a new leg (swing)
// @param leg           (int) Current leg value
// @returns             bool
startOfNewLeg(int leg)      => ta.change(leg) != 0

// @function            Identify whether the current level is the start of a new bearish leg (swing)
// @param leg           (int) Current leg value
// @returns             bool
startOfBearishLeg(int leg)  => ta.change(leg) == -1

// @function            Identify whether the current level is the start of a new bullish leg (swing)
// @param leg           (int) Current leg value
// @returns             bool
startOfBullishLeg(int leg)  => ta.change(leg) == +1

// @function            create a new label
// @param labelTime     bar time coordinate
// @param labelPrice    price coordinate
// @param tag           text to display
// @param labelColor    text color
// @param labelStyle    label style
// @returns             label ID
drawLabel(int labelTime, float labelPrice, string tag, color labelColor, string labelStyle) =>    
    var label l_abel = na

    if modeInput == PRESENT
        l_abel.delete()

    l_abel := label.new(chart.point.new(labelTime,na,labelPrice),tag,xloc.bar_time,color=color(na),textcolor=labelColor,style = labelStyle,size = size.small)

// @function            create a new line and label representing an EQH or EQL
// @param p_ivot        starting pivot
// @param level         price level of current pivot
// @param size          how many bars ago was the current pivot detected
// @param equalHigh     true for EQH, false for EQL
// @returns             label ID
drawEqualHighLow(pivot p_ivot, float level, int size, bool equalHigh) =>
    equalDisplay e_qualDisplay = equalHigh ? equalHighDisplay : equalLowDisplay
    
    string tag          = 'EQL'
    color equalColor    = swingBullishColor
    string labelStyle   = label.style_label_up

    if equalHigh
        tag         := 'EQH'
        equalColor  := swingBearishColor
        labelStyle  := label.style_label_down

    if modeInput == PRESENT
        line.delete(    e_qualDisplay.l_ine)
        label.delete(   e_qualDisplay.l_abel)
        
    e_qualDisplay.l_ine     := line.new(chart.point.new(p_ivot.barTime,na,p_ivot.currentLevel), chart.point.new(time[size],na,level), xloc = xloc.bar_time, color = equalColor, style = line.style_dotted)
    labelPosition           = math.round(0.5*(p_ivot.barIndex + bar_index - size))
    e_qualDisplay.l_abel    := label.new(chart.point.new(na,labelPosition,level), tag, xloc.bar_index, color = color(na), textcolor = equalColor, style = labelStyle, size = equalHighsLowsSizeInput)

// @function            store current structure and trailing swing points, and also display swing points and equal highs/lows
// @param size          (int) structure size
// @param equalHighLow  (bool) true for displaying current highs/lows
// @param internal      (bool) true for getting internal structures
// @returns             label ID
getCurrentStructure(int size,bool equalHighLow = false, bool internal = false) =>        
    currentLeg              = leg(size)
    newPivot                = startOfNewLeg(currentLeg)
    pivotLow                = startOfBullishLeg(currentLeg)
    pivotHigh               = startOfBearishLeg(currentLeg)

    if newPivot
        if pivotLow
            pivot p_ivot    = equalHighLow ? equalLow : internal ? internalLow : swingLow    

            if equalHighLow and math.abs(p_ivot.currentLevel - low[size]) < equalHighsLowsThresholdInput * atrMeasure                
                drawEqualHighLow(p_ivot, low[size], size, false)
                currentAlerts.equalLows := true

            p_ivot.lastLevel    := p_ivot.currentLevel
            p_ivot.currentLevel := low[size]
            p_ivot.crossed      := false
            p_ivot.barTime      := time[size]
            p_ivot.barIndex     := bar_index[size]

            if not equalHighLow and not internal
                trailing.bottom         := p_ivot.currentLevel
                trailing.barTime        := p_ivot.barTime
                trailing.barIndex       := p_ivot.barIndex
                trailing.lastBottomTime := p_ivot.barTime

            if showSwingsInput and not internal and not equalHighLow
                drawLabel(time[size], p_ivot.currentLevel, p_ivot.currentLevel < p_ivot.lastLevel ? 'LL' : 'HL', swingBullishColor, label.style_label_up)            
        else
            pivot p_ivot = equalHighLow ? equalHigh : internal ? internalHigh : swingHigh

            if equalHighLow and math.abs(p_ivot.currentLevel - high[size]) < equalHighsLowsThresholdInput * atrMeasure
                drawEqualHighLow(p_ivot,high[size],size,true)
                currentAlerts.equalHighs := true               

            p_ivot.lastLevel    := p_ivot.currentLevel
            p_ivot.currentLevel := high[size]
            p_ivot.crossed      := false
            p_ivot.barTime      := time[size]
            p_ivot.barIndex     := bar_index[size]

            if not equalHighLow and not internal
                trailing.top            := p_ivot.currentLevel
                trailing.barTime        := p_ivot.barTime
                trailing.barIndex       := p_ivot.barIndex
                trailing.lastTopTime    := p_ivot.barTime

            if showSwingsInput and not internal and not equalHighLow
                drawLabel(time[size], p_ivot.currentLevel, p_ivot.currentLevel > p_ivot.lastLevel ? 'HH' : 'LH', swingBearishColor, label.style_label_down)
                
// @function                draw line and label representing a structure
// @param p_ivot            base pivot point
// @param tag               test to display
// @param structureColor    base color
// @param lineStyle         line style
// @param labelStyle        label style
// @param labelSize         text size
// @returns                 label ID
drawStructure(pivot p_ivot, string tag, color structureColor, string lineStyle, string labelStyle, string labelSize) =>    
    var line l_ine      = line.new(na,na,na,na,xloc = xloc.bar_time)
    var label l_abel    = label.new(na,na)

    if modeInput == PRESENT
        l_ine.delete()
        l_abel.delete()

    l_ine   := line.new(chart.point.new(p_ivot.barTime,na,p_ivot.currentLevel), chart.point.new(time,na,p_ivot.currentLevel), xloc.bar_time, color=structureColor, style=lineStyle)
    l_abel  := label.new(chart.point.new(na,math.round(0.5*(p_ivot.barIndex+bar_index)),p_ivot.currentLevel), tag, xloc.bar_index, color=color(na), textcolor=structureColor, style=labelStyle, size = labelSize)

// @function            delete order blocks
// @param internal      true for internal order blocks
// @returns             orderBlock ID
deleteOrderBlocks(bool internal = false) =>
    array<orderBlock> orderBlocks = internal ? internalOrderBlocks : swingOrderBlocks

    for [index,eachOrderBlock] in orderBlocks
        bool crossedOderBlock = false
        
        if bearishOrderBlockMitigationSource > eachOrderBlock.barHigh and eachOrderBlock.bias == BEARISH
            crossedOderBlock := true
            if internal
                currentAlerts.internalBearishOrderBlock := true
            else
                currentAlerts.swingBearishOrderBlock    := true
        else if bullishOrderBlockMitigationSource < eachOrderBlock.barLow and eachOrderBlock.bias == BULLISH
            crossedOderBlock := true
            if internal
                currentAlerts.internalBullishOrderBlock := true
            else
                currentAlerts.swingBullishOrderBlock    := true
        if crossedOderBlock                    
            orderBlocks.remove(index)            

// @function            fetch and store order blocks
// @param p_ivot        base pivot point
// @param internal      true for internal order blocks
// @param bias          BULLISH or BEARISH
// @returns             void
storeOrdeBlock(pivot p_ivot,bool internal = false,int bias) =>
    if (not internal and showSwingOrderBlocksInput) or (internal and showInternalOrderBlocksInput)

        array<float> a_rray = na
        int parsedIndex = na

        if bias == BEARISH
            a_rray      := parsedHighs.slice(p_ivot.barIndex,bar_index)
            parsedIndex := p_ivot.barIndex + a_rray.indexof(a_rray.max())  
        else
            a_rray      := parsedLows.slice(p_ivot.barIndex,bar_index)
            parsedIndex := p_ivot.barIndex + a_rray.indexof(a_rray.min())                        

        orderBlock o_rderBlock          = orderBlock.new(parsedHighs.get(parsedIndex), parsedLows.get(parsedIndex), times.get(parsedIndex),bias)
        array<orderBlock> orderBlocks   = internal ? internalOrderBlocks : swingOrderBlocks
        
        if orderBlocks.size() >= 100
            orderBlocks.pop()
        orderBlocks.unshift(o_rderBlock)

// @function            draw order blocks as boxes
// @param internal      true for internal order blocks
// @returns             void
drawOrderBlocks(bool internal = false) =>        
    array<orderBlock> orderBlocks = internal ? internalOrderBlocks : swingOrderBlocks
    orderBlocksSize = orderBlocks.size()

    if orderBlocksSize > 0        
        maxOrderBlocks                      = internal ? internalOrderBlocksSizeInput : swingOrderBlocksSizeInput
        array<orderBlock> parsedOrdeBlocks  = orderBlocks.slice(0, math.min(maxOrderBlocks,orderBlocksSize))
        array<box> b_oxes                   = internal ? internalOrderBlocksBoxes : swingOrderBlocksBoxes        

        for [index,eachOrderBlock] in parsedOrdeBlocks
            orderBlockColor = styleInput == MONOCHROME ? (eachOrderBlock.bias == BEARISH ? color.new(MONO_BEARISH,80) : color.new(MONO_BULLISH,80)) : internal ? (eachOrderBlock.bias == BEARISH ? internalBearishOrderBlockColor : internalBullishOrderBlockColor) : (eachOrderBlock.bias == BEARISH ? swingBearishOrderBlockColor : swingBullishOrderBlockColor)

            box b_ox        = b_oxes.get(index)
            b_ox.set_top_left_point(    chart.point.new(eachOrderBlock.barTime,na,eachOrderBlock.barHigh))
            b_ox.set_bottom_right_point(chart.point.new(last_bar_time,na,eachOrderBlock.barLow))        
            b_ox.set_border_color(      internal ? na : orderBlockColor)
            b_ox.set_bgcolor(           orderBlockColor)

// @function            detect and draw structures, also detect and store order blocks
// @param internal      true for internal structures or order blocks
// @returns             void
displayStructure(bool internal = false) =>
    var bullishBar = true
    var bearishBar = true

    if internalFilterConfluenceInput
        bullishBar := high - math.max(close, open) > math.min(close, open - low)
        bearishBar := high - math.max(close, open) < math.min(close, open - low)
    
    pivot p_ivot    = internal ? internalHigh : swingHigh
    trend t_rend    = internal ? internalTrend : swingTrend

    lineStyle       = internal ? line.style_dashed : line.style_solid
    labelSize       = internal ? internalStructureSize : swingStructureSize

    extraCondition  = internal ? internalHigh.currentLevel != swingHigh.currentLevel and bullishBar : true
    bullishColor    = styleInput == MONOCHROME ? MONO_BULLISH : internal ? internalBullColorInput : swingBullColorInput

    if ta.crossover(close,p_ivot.currentLevel) and not p_ivot.crossed and extraCondition
        string tag = t_rend.bias == BEARISH ? CHOCH : BOS

        if internal
            currentAlerts.internalBullishCHoCH  := tag == CHOCH
            currentAlerts.internalBullishBOS    := tag == BOS
        else
            currentAlerts.swingBullishCHoCH     := tag == CHOCH
            currentAlerts.swingBullishBOS       := tag == BOS

        p_ivot.crossed  := true
        t_rend.bias     := BULLISH

        displayCondition = internal ? showInternalsInput and (showInternalBullInput == ALL or (showInternalBullInput == BOS and tag != CHOCH) or (showInternalBullInput == CHOCH and tag == CHOCH)) : showStructureInput and (showSwingBullInput == ALL or (showSwingBullInput == BOS and tag != CHOCH) or (showSwingBullInput == CHOCH and tag == CHOCH))

        if displayCondition                        
            drawStructure(p_ivot,tag,bullishColor,lineStyle,label.style_label_down,labelSize)

        if (internal and showInternalOrderBlocksInput) or (not internal and showSwingOrderBlocksInput)
            storeOrdeBlock(p_ivot,internal,BULLISH)

    p_ivot          := internal ? internalLow : swingLow    
    extraCondition  := internal ? internalLow.currentLevel != swingLow.currentLevel and bearishBar : true
    bearishColor    = styleInput == MONOCHROME ? MONO_BEARISH : internal ? internalBearColorInput : swingBearColorInput

    if ta.crossunder(close,p_ivot.currentLevel) and not p_ivot.crossed and extraCondition
        string tag = t_rend.bias == BULLISH ? CHOCH : BOS

        if internal
            currentAlerts.internalBearishCHoCH  := tag == CHOCH
            currentAlerts.internalBearishBOS    := tag == BOS
        else
            currentAlerts.swingBearishCHoCH     := tag == CHOCH
            currentAlerts.swingBearishBOS       := tag == BOS

        p_ivot.crossed := true
        t_rend.bias := BEARISH

        displayCondition = internal ? showInternalsInput and (showInternalBearInput == ALL or (showInternalBearInput == BOS and tag != CHOCH) or (showInternalBearInput == CHOCH and tag == CHOCH)) : showStructureInput and (showSwingBearInput == ALL or (showSwingBearInput == BOS and tag != CHOCH) or (showSwingBearInput == CHOCH and tag == CHOCH))
        
        if displayCondition                        
            drawStructure(p_ivot,tag,bearishColor,lineStyle,label.style_label_up,labelSize)

        if (internal and showInternalOrderBlocksInput) or (not internal and showSwingOrderBlocksInput)
            storeOrdeBlock(p_ivot,internal,BEARISH)

// @function            draw one fair value gap box (each fair value gap has two boxes)
// @param leftTime      left time coordinate
// @param rightTime     right time coordinate
// @param topPrice      top price level
// @param bottomPrice   bottom price level
// @param boxColor      box color
// @returns             box ID
fairValueGapBox(leftTime,rightTime,topPrice,bottomPrice,boxColor) => box.new(chart.point.new(leftTime,na,topPrice),chart.point.new(rightTime + fairValueGapsExtendInput * (time-time[1]),na,bottomPrice), xloc=xloc.bar_time, border_color = boxColor, bgcolor = boxColor)

// @function            delete fair value gaps
// @returns             fairValueGap ID
deleteFairValueGaps() =>
    for [index,eachFairValueGap] in fairValueGaps
        if (low < eachFairValueGap.bottom and eachFairValueGap.bias == BULLISH) or (high > eachFairValueGap.top and eachFairValueGap.bias == BEARISH)
            eachFairValueGap.topBox.delete()
            eachFairValueGap.bottomBox.delete()
            fairValueGaps.remove(index)
    
// @function            draw fair value gaps
// @returns             fairValueGap ID
drawFairValueGaps() => 
    [lastClose, lastOpen, lastTime, currentHigh, currentLow, currentTime, last2High, last2Low] = request.security(syminfo.tickerid, fairValueGapsTimeframeInput, [close[1], open[1], time[1], high[0], low[0], time[0], high[2], low[2]],lookahead = barmerge.lookahead_on)

    barDeltaPercent     = (lastClose - lastOpen) / (lastOpen * 100)
    newTimeframe        = timeframe.change(fairValueGapsTimeframeInput)
    threshold           = fairValueGapsThresholdInput ? ta.cum(math.abs(newTimeframe ? barDeltaPercent : 0)) / bar_index * 2 : 0

    bullishFairValueGap = currentLow > last2High and lastClose > last2High and barDeltaPercent > threshold and newTimeframe
    bearishFairValueGap = currentHigh < last2Low and lastClose < last2Low and -barDeltaPercent > threshold and newTimeframe

    if bullishFairValueGap
        currentAlerts.bullishFairValueGap := true
        fairValueGaps.unshift(fairValueGap.new(currentLow,last2High,BULLISH,fairValueGapBox(lastTime,currentTime,currentLow,math.avg(currentLow,last2High),fairValueGapBullishColor),fairValueGapBox(lastTime,currentTime,math.avg(currentLow,last2High),last2High,fairValueGapBullishColor)))
    if bearishFairValueGap
        currentAlerts.bearishFairValueGap := true
        fairValueGaps.unshift(fairValueGap.new(currentHigh,last2Low,BEARISH,fairValueGapBox(lastTime,currentTime,currentHigh,math.avg(currentHigh,last2Low),fairValueGapBearishColor),fairValueGapBox(lastTime,currentTime,math.avg(currentHigh,last2Low),last2Low,fairValueGapBearishColor)))

// @function            get line style from string
// @param style         line style
// @returns             string
getStyle(string style) =>
    switch style
        SOLID => line.style_solid
        DASHED => line.style_dashed
        DOTTED => line.style_dotted

// @function            draw MultiTimeFrame levels
// @param timeframe     base timeframe
// @param sameTimeframe true if chart timeframe is same as base timeframe
// @param style         line style
// @param levelColor    line and text color
// @returns             void
drawLevels(string timeframe, bool sameTimeframe, string style, color levelColor) =>
    [topLevel, bottomLevel, leftTime, rightTime] = request.security(syminfo.tickerid, timeframe, [high[1], low[1], time[1], time],lookahead = barmerge.lookahead_on)

    float parsedTop         = sameTimeframe ? high : topLevel
    float parsedBottom      = sameTimeframe ? low : bottomLevel    

    int parsedLeftTime      = sameTimeframe ? time : leftTime
    int parsedRightTime     = sameTimeframe ? time : rightTime

    int parsedTopTime       = time
    int parsedBottomTime    = time

    if not sameTimeframe
        int leftIndex               = times.binary_search_rightmost(parsedLeftTime)
        int rightIndex              = times.binary_search_rightmost(parsedRightTime)

        array<int> timeArray        = times.slice(leftIndex,rightIndex)
        array<float> topArray       = highs.slice(leftIndex,rightIndex)
        array<float> bottomArray    = lows.slice(leftIndex,rightIndex)

        parsedTopTime               := timeArray.size() > 0 ? timeArray.get(topArray.indexof(topArray.max())) : initialTime
        parsedBottomTime            := timeArray.size() > 0 ? timeArray.get(bottomArray.indexof(bottomArray.min())) : initialTime

    var line topLine        = line.new(na, na, na, na, xloc = xloc.bar_time, color = levelColor, style = getStyle(style))
    var line bottomLine     = line.new(na, na, na, na, xloc = xloc.bar_time, color = levelColor, style = getStyle(style))
    var label topLabel      = label.new(na, na, xloc = xloc.bar_time, text = str.format('P{0}H',timeframe), color=color(na), textcolor = levelColor, size = size.small, style = label.style_label_left)
    var label bottomLabel   = label.new(na, na, xloc = xloc.bar_time, text = str.format('P{0}L',timeframe), color=color(na), textcolor = levelColor, size = size.small, style = label.style_label_left)

    topLine.set_first_point(    chart.point.new(parsedTopTime,na,parsedTop))
    topLine.set_second_point(   chart.point.new(last_bar_time + 20 * (time-time[1]),na,parsedTop))   
    topLabel.set_point(         chart.point.new(last_bar_time + 20 * (time-time[1]),na,parsedTop))

    bottomLine.set_first_point( chart.point.new(parsedBottomTime,na,parsedBottom))    
    bottomLine.set_second_point(chart.point.new(last_bar_time + 20 * (time-time[1]),na,parsedBottom))
    bottomLabel.set_point(      chart.point.new(last_bar_time + 20 * (time-time[1]),na,parsedBottom))

// @function            true if chart timeframe is higher than provided timeframe
// @param timeframe     timeframe to check
// @returns             bool
higherTimeframe(string timeframe) => timeframe.in_seconds() > timeframe.in_seconds(timeframe)

// @function            update trailing swing points
// @returns             int
updateTrailingExtremes() =>
    trailing.top            := math.max(high,trailing.top)
    trailing.lastTopTime    := trailing.top == high ? time : trailing.lastTopTime
    trailing.bottom         := math.min(low,trailing.bottom)
    trailing.lastBottomTime := trailing.bottom == low ? time : trailing.lastBottomTime

// @function            draw trailing swing points
// @returns             void
drawHighLowSwings() =>
    var line topLine        = line.new(na, na, na, na, color = swingBearishColor, xloc = xloc.bar_time)
    var line bottomLine     = line.new(na, na, na, na, color = swingBullishColor, xloc = xloc.bar_time)
    var label topLabel      = label.new(na, na, color=color(na), textcolor = swingBearishColor, xloc = xloc.bar_time, style = label.style_label_down, size = size.tiny)
    var label bottomLabel   = label.new(na, na, color=color(na), textcolor = swingBullishColor, xloc = xloc.bar_time, style = label.style_label_up, size = size.tiny)

    rightTimeBar            = last_bar_time + 20 * (time - time[1])

    topLine.set_first_point(    chart.point.new(trailing.lastTopTime, na, trailing.top))
    topLine.set_second_point(   chart.point.new(rightTimeBar, na, trailing.top))
    topLabel.set_point(         chart.point.new(rightTimeBar, na, trailing.top))
    topLabel.set_text(          swingTrend.bias == BEARISH ? 'Strong High' : 'Weak High')

    bottomLine.set_first_point( chart.point.new(trailing.lastBottomTime, na, trailing.bottom))
    bottomLine.set_second_point(chart.point.new(rightTimeBar, na, trailing.bottom))
    bottomLabel.set_point(      chart.point.new(rightTimeBar, na, trailing.bottom))
    bottomLabel.set_text(       swingTrend.bias == BULLISH ? 'Strong Low' : 'Weak Low')

// @function            draw a zone with a label and a box
// @param labelLevel    price level for label
// @param labelIndex    bar index for label
// @param top           top price level for box
// @param bottom        bottom price level for box
// @param tag           text to display
// @param zoneColor     base color
// @param style         label style
// @returns             void
drawZone(float labelLevel, int labelIndex, float top, float bottom, string tag, color zoneColor, string style) =>
    var label l_abel    = label.new(na,na,text = tag, color=color(na),textcolor = zoneColor, style = style, size = size.small)
    var box b_ox        = box.new(na,na,na,na,bgcolor = color.new(zoneColor,80),border_color = color(na), xloc = xloc.bar_time)

    b_ox.set_top_left_point(    chart.point.new(trailing.barTime,na,top))
    b_ox.set_bottom_right_point(chart.point.new(last_bar_time,na,bottom))

    l_abel.set_point(           chart.point.new(na,labelIndex,labelLevel))

// @function            draw premium/discount zones
// @returns             void
drawPremiumDiscountZones() =>
    drawZone(trailing.top, math.round(0.5*(trailing.barIndex + last_bar_index)), trailing.top, 0.95*trailing.top + 0.05*trailing.bottom, 'Premium', premiumZoneColor, label.style_label_down)

    equilibriumLevel = math.avg(trailing.top, trailing.bottom)
    drawZone(equilibriumLevel, last_bar_index, 0.525*trailing.top + 0.475*trailing.bottom, 0.525*trailing.bottom + 0.475*trailing.top, 'Equilibrium', equilibriumZoneColorInput, label.style_label_left)

    drawZone(trailing.bottom, math.round(0.5*(trailing.barIndex + last_bar_index)), 0.95*trailing.bottom + 0.05*trailing.top, trailing.bottom, 'Discount', discountZoneColor, label.style_label_up)

//---------------------------------------------------------------------------------------------------------------------}
//MUTABLE VARIABLES & EXECUTION
//---------------------------------------------------------------------------------------------------------------------{
parsedOpen  = enableSmartMoneyInput and showTrendInput ? open : na
candleColor = internalTrend.bias == BULLISH ? swingBullishColor : swingBearishColor
plotcandle(parsedOpen,high,low,close,color = candleColor, wickcolor = candleColor, bordercolor = candleColor)

if enableSmartMoneyInput and (showHighLowSwingsInput or showPremiumDiscountZonesInput)
    updateTrailingExtremes()

    if showHighLowSwingsInput
        drawHighLowSwings()

    if showPremiumDiscountZonesInput
        drawPremiumDiscountZones()

if enableSmartMoneyInput and showFairValueGapsInput
    deleteFairValueGaps()

if enableSmartMoneyInput
    getCurrentStructure(swingsLengthInput,false)
    getCurrentStructure(5,false,true)

if enableSmartMoneyInput and showEqualHighsLowsInput
    getCurrentStructure(equalHighsLowsLengthInput,true)

if enableSmartMoneyInput and (showInternalsInput or showInternalOrderBlocksInput or showTrendInput)
    displayStructure(true)

if enableSmartMoneyInput and (showStructureInput or showSwingOrderBlocksInput or showHighLowSwingsInput)
    displayStructure()

if enableSmartMoneyInput and showInternalOrderBlocksInput
    deleteOrderBlocks(true)

if enableSmartMoneyInput and showSwingOrderBlocksInput
    deleteOrderBlocks()

if enableSmartMoneyInput and showFairValueGapsInput
    drawFairValueGaps()

if enableSmartMoneyInput and (barstate.islastconfirmedhistory or barstate.islast)
    if showInternalOrderBlocksInput        
        drawOrderBlocks(true)
        
    if showSwingOrderBlocksInput        
        drawOrderBlocks()

lastBarIndex    := currentBarIndex
currentBarIndex := bar_index
newBar          = currentBarIndex != lastBarIndex

if enableSmartMoneyInput and (barstate.islastconfirmedhistory or (barstate.isrealtime and newBar))
    if showDailyLevelsInput and not higherTimeframe('D')
        drawLevels('D',timeframe.isdaily,dailyLevelsStyleInput,dailyLevelsColorInput)

    if showWeeklyLevelsInput and not higherTimeframe('W')
        drawLevels('W',timeframe.isweekly,weeklyLevelsStyleInput,weeklyLevelsColorInput)

    if showMonthlyLevelsInput and not higherTimeframe('M')
        drawLevels('M',timeframe.ismonthly,monthlyLevelsStyleInput,monthlyLevelsColorInput)

//---------------------------------------------------------------------------------------------------------------------}
//ALERTS
//---------------------------------------------------------------------------------------------------------------------{
alertcondition(enableSmartMoneyInput and currentAlerts.internalBullishBOS,        'Internal Bullish BOS',         'Internal Bullish BOS formed')
alertcondition(enableSmartMoneyInput and currentAlerts.internalBullishCHoCH,      'Internal Bullish CHoCH',       'Internal Bullish CHoCH formed')
alertcondition(enableSmartMoneyInput and currentAlerts.internalBearishBOS,        'Internal Bearish BOS',         'Internal Bearish BOS formed')
alertcondition(enableSmartMoneyInput and currentAlerts.internalBearishCHoCH,      'Internal Bearish CHoCH',       'Internal Bearish CHoCH formed')

alertcondition(enableSmartMoneyInput and currentAlerts.swingBullishBOS,           'Bullish BOS',                  'Internal Bullish BOS formed')
alertcondition(enableSmartMoneyInput and currentAlerts.swingBullishCHoCH,         'Bullish CHoCH',                'Internal Bullish CHoCH formed')
alertcondition(enableSmartMoneyInput and currentAlerts.swingBearishBOS,           'Bearish BOS',                  'Bearish BOS formed')
alertcondition(enableSmartMoneyInput and currentAlerts.swingBearishCHoCH,         'Bearish CHoCH',                'Bearish CHoCH formed')

alertcondition(enableSmartMoneyInput and currentAlerts.internalBullishOrderBlock, 'Bullish Internal OB Breakout', 'Price broke bullish internal OB')
alertcondition(enableSmartMoneyInput and currentAlerts.internalBearishOrderBlock, 'Bearish Internal OB Breakout', 'Price broke bearish internal OB')
alertcondition(enableSmartMoneyInput and currentAlerts.swingBullishOrderBlock,    'Bullish Swing OB Breakout',    'Price broke bullish swing OB')
alertcondition(enableSmartMoneyInput and currentAlerts.swingBearishOrderBlock,    'Bearish Swing OB Breakout',    'Price broke bearish swing OB')

alertcondition(enableSmartMoneyInput and currentAlerts.equalHighs,                'Equal Highs',                  'Equal highs detected')
alertcondition(enableSmartMoneyInput and currentAlerts.equalLows,                 'Equal Lows',                   'Equal lows detected')

alertcondition(enableSmartMoneyInput and currentAlerts.bullishFairValueGap,       'Bullish FVG',                  'Bullish FVG formed')
alertcondition(enableSmartMoneyInput and currentAlerts.bearishFairValueGap,       'Bearish FVG',                  'Bearish FVG formed')

//---------------------------------------------------------------------------------------------------------------------}

// ===== MODULE 2: FIBFANS / PIVOTS / VOLUME =====

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
    if enableFibFansInput and _show
        style = _s == 'Solid' ? line.style_solid : _s == 'Dotted' ? line.style_dotted : line.style_dashed
        f_drawLineX(_x1, _y, _x2, _y, xloc.bar_time, extend.none, _c, style, _w)
        f_drawLabelX(_x2, _y, _lb, xloc.bar_time, yloc.price, #00000000, label.style_label_left, _c, size.normal, text.align_left, _tip + ' ' + _lb + '\n' + str.tostring(_y, format.mintick))

    if enableFibFansInput and f_crossingLevelX(close, _y) and _show
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



group_levels = '10. FibFans • HTF & Fans'
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

group_pick_a_pivot = '11. FibFans • Pivot Points'
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
i_isRange = input.bool(true, 'Price Range/Change Meter', group='13. FibFans • Price Range Meter', tooltip=tooltip_range)

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

if enableFibFansInput and barstate.islast
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

bgcolor(enableFibFansInput and i_isChangeHTF and ta.change(time(i_htf)) ? color.new(color.blue, 89) : na)

if enableFibFansInput and f_crossingLevelX(close, H)
    alert(syminfo.ticker + ' crossing previous high(' + str.tostring(histHL) + '), price - ' + str.tostring(H, format.mintick))
if enableFibFansInput and f_crossingLevelX(close, L)
    alert(syminfo.ticker + ' crossing previous low('  + str.tostring(histHL) + '), price - ' + str.tostring(L, format.mintick)) 

// ---------------------------------------------------------------------------------------------- //
// Pivot Point Calculations --------------------------------------------------------------------- //

[R6, R5, R4, R3, R2, R1, P, S1, S2, S3, S4, S5, S6] = f_get_pivot(O , H , L , C , O0, i_pickAPivot)
[r6, r5, r4, r3, r2, r1, p, s1, s2, s3, s4, s5, s6] = f_get_pivot(O0, H0, L0, C0, O0, i_pickAPivot)
CPR  = math.avg(H , L , C ), BC  = math.avg(H , L ), TC  = 2 * CPR  - BC
CPR0 = math.avg(H0, L0, C0), BC0 = math.avg(H0, L0), TC0 = 2 * CPR0 - BC0

plot(enableFibFansInput and i_isCPR ? CPR : na, 'Central Pivot Range' , ta.change(time(i_htf)) ? na : #4262ba)
plot(enableFibFansInput and i_isCPR ? BC  : na, 'Bottom Central Level', ta.change(time(i_htf)) ? na : #fa8072)
plot(enableFibFansInput and i_isCPR ? TC  : na, 'Top Central Level'   , ta.change(time(i_htf)) ? na : #9ef2e8)

when = enableFibFansInput and barstate.islast and i_isCPR
f_processPivotLevelX(when, time_x1, CPR, time_x2, #4262ba, 'Solid', 1, 'CPR', 'Central Pivot Range' )
f_processPivotLevelX(when, time_x1, BC , time_x2, #fa8072, 'Solid', 1, 'BC' , 'Bottom Central Level')
f_processPivotLevelX(when, time_x1, TC , time_x2, #9ef2e8, 'Solid', 1, 'TC' , 'Top Central Level'   )

when := enableFibFansInput and barstate.islast and i_isCPR and i_dispPVT and time_x2 - timenow < 3600000 * i_when
f_processPivotLevelX(when, time_x2, CPR0, time_x20, #4262ba, 'Solid', 1, 'CPR', 'Subsequent Central Pivot Range' )
f_processPivotLevelX(when, time_x2, BC0 , time_x20, #fa8072, 'Solid', 1, 'BC' , 'Subsequent Bottom Central Level')
f_processPivotLevelX(when, time_x2, TC0 , time_x20, #9ef2e8, 'Solid', 1, 'TC' , 'Subsequent Top Central Level'   )

when := enableFibFansInput and barstate.islast and i_isPivot
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

when := enableFibFansInput and barstate.islast and i_isPivot and CPR != P and P > 0
f_processPivotLevelX(when and i_show_p, time_x1, P, time_x2, i_color_p, i_style_p, i_width_p, 'P', i_pickAPivot)

when := enableFibFansInput and barstate.islast and i_isPivot and i_dispPVT and time_x2 - timenow < 3600000 * i_when
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

when := enableFibFansInput and barstate.islast and i_isPivot and i_dispPVT and CPR0 != p and p > 0 and time_x2 - timenow < 3600000 * i_when
f_processPivotLevelX(when and i_show_p, time_x2, P, time_x20, i_color_p, i_style_p, i_width_p, 'P', 'Subsequent ' + i_pickAPivot)

// ---------------------------------------------------------------------------------------------- //
// Fib Fans Calculations ------------------------------------------------------------------------ //

f_processFanLevels(_show, _level, _color, _width, _style, _line) =>
    if enableFibFansInput and _show
        style = _style == 'Solid' ? line.style_solid : _style == 'Dotted' ? line.style_dotted : line.style_dashed

        if not i_isLeftFan
            f_drawLineX(i_prev ? time_x1 : time_x11, H, time_x2, H + range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)
            f_drawLineX(i_prev ? time_x1 : time_x11, H, time_x2, H - range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)
            f_drawLineX(i_prev ? time_x1 : time_x11, L, time_x2, L + range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)
            f_drawLineX(i_prev ? time_x1 : time_x11, L, time_x2, L - range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)

        if enableFibFansInput and f_crossingLevelX(close, f_getPrice(i_prev ? time_x1 : time_x11, H, time_x2, H + range_1 * _level)) and not i_isLeftFan
            alert('AutoFibFan Left Upper: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_prev ? time_x1 : time_x11, H, time_x2, H + range_1 * _level), format.mintick))
        if enableFibFansInput and f_crossingLevelX(close, f_getPrice(i_prev ? time_x1 : time_x11, H, time_x2, H - range_1 * _level)) and not i_isLeftFan
            alert('AutoFibFan Left Upper: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_prev ? time_x1 : time_x11, H, time_x2, H - range_1 * _level), format.mintick))
        if enableFibFansInput and f_crossingLevelX(close, f_getPrice(i_prev ? time_x1 : time_x11, L, time_x2, L + range_1 * _level)) and not i_isLeftFan
            alert('AutoFibFan Left Lower: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_prev ? time_x1 : time_x11, L, time_x2, L + range_1 * _level), format.mintick))
        if enableFibFansInput and f_crossingLevelX(close, f_getPrice(i_prev ? time_x1 : time_x11, L, time_x2, L - range_1 * _level)) and not i_isLeftFan
            alert('AutoFibFan Left Lower: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_prev ? time_x1 : time_x11, L, time_x2, L - range_1 * _level), format.mintick))
            
        if not i_isRightFan 
            f_drawLineX(i_back ? time_x2 : time_x20, H, time_x1, H + range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)
            f_drawLineX(i_back ? time_x2 : time_x20, H, time_x1, H - range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)
            f_drawLineX(i_back ? time_x2 : time_x20, L, time_x1, L + range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)
            f_drawLineX(i_back ? time_x2 : time_x20, L, time_x1, L - range_1 * _level, xloc.bar_time, extend.none, _color, style, _width)

        if enableFibFansInput and f_crossingLevelX(close, f_getPrice(i_back ? time_x2 : time_x20, H, time_x1, H + range_1 * _level)) and not i_isRightFan
            alert('AutoFibFan Right Upper: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_back ? time_x2 : time_x20, H, time_x1, H + range_1 * _level), format.mintick))
        if enableFibFansInput and f_crossingLevelX(close, f_getPrice(i_back ? time_x2 : time_x20, H, time_x1, H - range_1 * _level)) and not i_isRightFan
            alert('AutoFibFan Right Upper: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_back ? time_x2 : time_x20, H, time_x1, H - range_1 * _level), format.mintick))
        if enableFibFansInput and f_crossingLevelX(close, f_getPrice(i_back ? time_x2 : time_x20, L, time_x1, L + range_1 * _level)) and not i_isRightFan
            alert('AutoFibFan Right Lower: ' + syminfo.ticker + ' crossing level ' + str.tostring(_level) + ', price - ' + str.tostring(f_getPrice(i_back ? time_x2 : time_x20, L, time_x1, L + range_1 * _level), format.mintick))
        if enableFibFansInput and f_crossingLevelX(close, f_getPrice(i_back ? time_x2 : time_x20, L, time_x1, L - range_1 * _level)) and not i_isRightFan
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
group_vol_vol = '12. FibFans • Volume Add-Ons'

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
plotchar(enableFibFansInput and i_vSpikeLb and nzVolume ? exhaustVol : na, 'Exhaustion Bar', '🚦', location.abovebar, size=size.tiny)
alertcondition(enableFibFansInput and ta.crossover(nzVolume, i_vSMA * i_vSpikeThresh), 'Volume Spikes', 'sign of exhaustion, huge volume increase detected\n{{exchange}}:{{ticker}}->\nOpen = {{open}}, Current = {{close}},\nTime = {{time}}')

// ---------------------------------------------------------------------------------------------- //
// Volume Weighted Colored Bars ----------------------------------------------------------------- //

i_vwcb = input.bool(true, '', inline='VWC', group=group_vol_vol, tooltip=tooltip_volume_weighted_colored_bars)
i_vwcbHighThresh = input.float(1.618, 'VWCB :   High ', minval=1., step=.1, inline='VWC', group=group_vol_vol)
i_vwcbLowThresh = input.float(0.618, ' Low', minval=.1, step=.1, inline='VWC', group=group_vol_vol)

vwcbCol = nzVolume > i_vSMA * i_vwcbHighThresh ? close > open ? #006400 : #910000 : nzVolume < i_vSMA * i_vwcbLowThresh ? close < open ? #FF9800 : #7FFFD4 : na

barcolor(enableFibFansInput and i_vwcb and nzVolume ? vwcbCol : na, title='Volume Weighted Colored Bars')

// Voloume / Volatility AddOns
// ══════════════════════════════════════════════════════════════════════════════════════════════════ //

// ══════════════════════════════════════════════════════════════════════════════════════════════════ //
// Price Range Meter AddOn

f_atr(_length) =>
    ta.atr(_length)
atr = request.security(syminfo.tickerid, i_htf, f_atr(14))

if enableFibFansInput and i_isRange
    l = ta.valuewhen(ta.change(time(i_htf)), bar_index, 0) - ta.valuewhen(ta.change(time(i_htf)), bar_index, 1) 
    a = atr / 5
    t = time
    dif = time_x1 - time_x11
    tco = math.round(ta.change(t))
    f = math.min(L, L0)
    //------------------------------------------------------------------------------

    highVolatility = H0 - L0 > atr * 1.618

    if enableFibFansInput and barstate.islast
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


// ===== MODULE 3: EXTRA SMA / VOLUME MOMENTUM / ICHIMOKU =====

PRO_MTF_GROUP = '14. Pro Overlay • Multi-Timeframe'
useSingleOverlayTfInput       = input.bool(false, 'Use One Timeframe For All Pro Overlays', group = PRO_MTF_GROUP)
proOverlayTimeframeInput      = input.timeframe('', 'Master Pro Overlay Timeframe', group = PRO_MTF_GROUP)
smaTimeframeInput             = input.timeframe('', 'SMA Timeframe', group = PRO_MTF_GROUP)
volumeMomentumTimeframeInput  = input.timeframe('', 'Volume Momentum Timeframe', group = PRO_MTF_GROUP)
ichimokuTimeframeInput        = input.timeframe('', 'Ichimoku Timeframe', group = PRO_MTF_GROUP)

proOverlayTfFinal       = proOverlayTimeframeInput == '' ? timeframe.period : proOverlayTimeframeInput
smaTfFinal              = useSingleOverlayTfInput ? proOverlayTfFinal : (smaTimeframeInput == '' ? timeframe.period : smaTimeframeInput)
volumeMomentumTfFinal   = useSingleOverlayTfInput ? proOverlayTfFinal : (volumeMomentumTimeframeInput == '' ? timeframe.period : volumeMomentumTimeframeInput)
ichimokuTfFinal         = useSingleOverlayTfInput ? proOverlayTfFinal : (ichimokuTimeframeInput == '' ? timeframe.period : ichimokuTimeframeInput)
chartTfSeconds          = timeframe.in_seconds()
smaTfSeconds            = timeframe.in_seconds(smaTfFinal)
volumeMomentumTfSeconds = timeframe.in_seconds(volumeMomentumTfFinal)
ichimokuTfSeconds       = timeframe.in_seconds(ichimokuTfFinal)

EXTRA_SMA_GROUP = '15. Pro Overlay • SMA Pack'
showSma21Input   = input.bool(true,  'Show SMA 21',  group = EXTRA_SMA_GROUP, inline = 'sma21')
sma21ColorInput  = input.color(color.new(#00BCD4, 0), '', group = EXTRA_SMA_GROUP, inline = 'sma21')
showSma34Input   = input.bool(true,  'Show SMA 34',  group = EXTRA_SMA_GROUP, inline = 'sma34')
sma34ColorInput  = input.color(color.new(#8BC34A, 0), '', group = EXTRA_SMA_GROUP, inline = 'sma34')
showSma55Input   = input.bool(true,  'Show SMA 55',  group = EXTRA_SMA_GROUP, inline = 'sma55')
sma55ColorInput  = input.color(color.new(#FF9800, 0), '', group = EXTRA_SMA_GROUP, inline = 'sma55')
showSma200Input  = input.bool(true,  'Show SMA 200', group = EXTRA_SMA_GROUP, inline = 'sma200')
sma200ColorInput = input.color(color.new(#E91E63, 0), '', group = EXTRA_SMA_GROUP, inline = 'sma200')

[sma21Value, sma34Value, sma55Value, sma200Value] = request.security(syminfo.tickerid, smaTfFinal, [ta.sma(close, 21), ta.sma(close, 34), ta.sma(close, 55), ta.sma(close, 200)])

showSma21Final  = enableProOverlaysInput and showSma21Input and performanceModeInput != 'Lite'
showSma34Final  = enableProOverlaysInput and showSma34Input and performanceModeInput == 'Full'
showSma55Final  = enableProOverlaysInput and showSma55Input
showSma200Final = enableProOverlaysInput and showSma200Input

plot(showSma21Final  ? sma21Value  : na, 'SMA 21',  color = sma21ColorInput,  linewidth = 2)
plot(showSma34Final  ? sma34Value  : na, 'SMA 34',  color = sma34ColorInput,  linewidth = 2)
plot(showSma55Final  ? sma55Value  : na, 'SMA 55',  color = sma55ColorInput,  linewidth = 2)
plot(showSma200Final ? sma200Value : na, 'SMA 200', color = sma200ColorInput, linewidth = 2)

VOLUME_MOMENTUM_GROUP = '16. Pro Overlay • Volume Momentum'
showVolumeMomentumInput       = input.bool(true, 'Show Volume Momentum Signals', group = VOLUME_MOMENTUM_GROUP)
volumeMomentumLengthInput     = input.int(20, 'Volume SMA Length', minval = 1, group = VOLUME_MOMENTUM_GROUP)
volumeMomentumPriceLenInput   = input.int(5, 'Price Momentum Length', minval = 1, group = VOLUME_MOMENTUM_GROUP)
volumeMomentumThresholdInput  = input.float(1.5, 'Volume Strength Multiplier', minval = 0.1, step = 0.1, group = VOLUME_MOMENTUM_GROUP)
volumeMomentumBullColorInput  = input.color(color.new(#00E676, 0), 'Bullish Signal Color', group = VOLUME_MOMENTUM_GROUP)
volumeMomentumBearColorInput  = input.color(color.new(#FF5252, 0), 'Bearish Signal Color', group = VOLUME_MOMENTUM_GROUP)

volumeMomentumLengthFinal = profilePresetInput == 'Scalping' ? 13 : profilePresetInput == 'Intraday' ? 20 : profilePresetInput == 'Swing' ? 34 : volumeMomentumLengthInput
priceMomentumLengthFinal  = profilePresetInput == 'Scalping' ? 3  : profilePresetInput == 'Intraday' ? 5  : profilePresetInput == 'Swing' ? 8  : volumeMomentumPriceLenInput
volumeThresholdFinal      = profilePresetInput == 'Scalping' ? 1.3 : profilePresetInput == 'Intraday' ? 1.5 : profilePresetInput == 'Swing' ? 1.8 : volumeMomentumThresholdInput
showVolumeMomentumFinal   = enableProOverlaysInput and showVolumeMomentumInput and performanceModeInput != 'Lite'

[vmOpen, vmClose, vmVolume, vmVolumeBase, vmPriceMomentum] = request.security(syminfo.tickerid, volumeMomentumTfFinal, [open, close, volume, ta.sma(volume, volumeMomentumLengthFinal), ta.roc(close, priceMomentumLengthFinal)])
volumeStrengthValue = vmVolumeBase > 0 ? vmVolume / vmVolumeBase : 0.0
volumeMomentumSignalBar = volumeMomentumTfSeconds <= chartTfSeconds ? true : ta.change(time(volumeMomentumTfFinal))
bullishVolumeMomentum = showVolumeMomentumFinal and volumeMomentumSignalBar and vmClose > vmOpen and vmPriceMomentum > 0 and volumeStrengthValue >= volumeThresholdFinal
bearishVolumeMomentum = showVolumeMomentumFinal and volumeMomentumSignalBar and vmClose < vmOpen and vmPriceMomentum < 0 and volumeStrengthValue >= volumeThresholdFinal

plotshape(bullishVolumeMomentum, title = 'Bullish Volume Momentum', style = shape.triangleup, location = location.belowbar, color = volumeMomentumBullColorInput, size = size.tiny, text = 'VM+')
plotshape(bearishVolumeMomentum, title = 'Bearish Volume Momentum', style = shape.triangledown, location = location.abovebar, color = volumeMomentumBearColorInput, size = size.tiny, text = 'VM-')

ICHIMOKU_GROUP = '17. Pro Overlay • Ichimoku'
showIchimokuInput            = input.bool(true, 'Show Ichimoku', group = ICHIMOKU_GROUP)
showIchimokuTenkanInput      = input.bool(true, 'Tenkan', group = ICHIMOKU_GROUP, inline = 'ich1')
showIchimokuKijunInput       = input.bool(true, 'Kijun',  group = ICHIMOKU_GROUP, inline = 'ich1')
showIchimokuChikouInput      = input.bool(false, 'Chikou', group = ICHIMOKU_GROUP, inline = 'ich1')
showIchimokuCloudInput       = input.bool(true, 'Cloud', group = ICHIMOKU_GROUP)
ichimokuConversionLenInput   = input.int(9,  'Tenkan Length', minval = 1, group = ICHIMOKU_GROUP)
ichimokuBaseLenInput         = input.int(26, 'Kijun Length', minval = 1, group = ICHIMOKU_GROUP)
ichimokuSpanBLenInput        = input.int(52, 'Senkou B Length', minval = 1, group = ICHIMOKU_GROUP)
ichimokuDisplacementInput    = input.int(26, 'Displacement', minval = 1, group = ICHIMOKU_GROUP)
ichimokuTenkanColorInput     = input.color(color.new(#2962FF, 0), 'Tenkan Color', group = ICHIMOKU_GROUP)
ichimokuKijunColorInput      = input.color(color.new(#FF6D00, 0), 'Kijun Color', group = ICHIMOKU_GROUP)
ichimokuChikouColorInput     = input.color(color.new(#AB47BC, 0), 'Chikou Color', group = ICHIMOKU_GROUP)
ichimokuBullCloudColorInput  = input.color(color.new(#00C853, 82), 'Bull Cloud Color', group = ICHIMOKU_GROUP)
ichimokuBearCloudColorInput  = input.color(color.new(#D50000, 82), 'Bear Cloud Color', group = ICHIMOKU_GROUP)

ichimokuDonchian(_length) =>
    math.avg(ta.lowest(_length), ta.highest(_length))

[ichimokuTenkan, ichimokuKijun, ichimokuSpanA, ichimokuSpanB, ichimokuChikou] = request.security(syminfo.tickerid, ichimokuTfFinal, [ichimokuDonchian(ichimokuConversionLenInput), ichimokuDonchian(ichimokuBaseLenInput), math.avg(ichimokuDonchian(ichimokuConversionLenInput), ichimokuDonchian(ichimokuBaseLenInput)), ichimokuDonchian(ichimokuSpanBLenInput), close])
showIchimokuBaseFinal   = enableProOverlaysInput and showIchimokuInput
showIchimokuCloudFinal  = enableProOverlaysInput and showIchimokuInput and showIchimokuCloudInput and performanceModeInput != 'Lite'
showIchimokuChikouFinal = enableProOverlaysInput and showIchimokuInput and showIchimokuChikouInput and performanceModeInput == 'Full'
ichimokuSignalBar       = ichimokuTfSeconds <= chartTfSeconds ? true : ta.change(time(ichimokuTfFinal))

plot(showIchimokuBaseFinal and showIchimokuTenkanInput ? ichimokuTenkan : na, 'Ichimoku Tenkan', color = ichimokuTenkanColorInput, linewidth = 2)
plot(showIchimokuBaseFinal and showIchimokuKijunInput  ? ichimokuKijun  : na, 'Ichimoku Kijun',  color = ichimokuKijunColorInput,  linewidth = 2)
plot(showIchimokuChikouFinal ? ichimokuChikou : na, 'Ichimoku Chikou', color = ichimokuChikouColorInput, linewidth = 1, offset = -ichimokuDisplacementInput)
ichimokuSpanAPlot = plot(showIchimokuCloudFinal ? ichimokuSpanA : na, 'Ichimoku Senkou A', color = color.new(#00C853, 0), linewidth = 1, offset = ichimokuDisplacementInput)
ichimokuSpanBPlot = plot(showIchimokuCloudFinal ? ichimokuSpanB : na, 'Ichimoku Senkou B', color = color.new(#D50000, 0), linewidth = 1, offset = ichimokuDisplacementInput)
fill(ichimokuSpanAPlot, ichimokuSpanBPlot, color = showIchimokuCloudFinal ? (ichimokuSpanA >= ichimokuSpanB ? ichimokuBullCloudColorInput : ichimokuBearCloudColorInput) : na, title = 'Ichimoku Cloud')

// Pro overlay alerts
smaBullCross2155 = ta.crossover(sma21Value, sma55Value)
smaBearCross2155 = ta.crossunder(sma21Value, sma55Value)
priceCrossAboveSma200 = ta.crossover(close, sma200Value)
priceCrossBelowSma200 = ta.crossunder(close, sma200Value)
ichimokuBullTkCross = ichimokuSignalBar and ta.crossover(ichimokuTenkan, ichimokuKijun)
ichimokuBearTkCross = ichimokuSignalBar and ta.crossunder(ichimokuTenkan, ichimokuKijun)

alertcondition(enableProOverlaysInput and smaBullCross2155, 'SMA 21/55 Bullish Cross', 'SMA 21 crossed above SMA 55')
alertcondition(enableProOverlaysInput and smaBearCross2155, 'SMA 21/55 Bearish Cross', 'SMA 21 crossed below SMA 55')
alertcondition(enableProOverlaysInput and priceCrossAboveSma200, 'Price Cross Above SMA 200', 'Price crossed above SMA 200')
alertcondition(enableProOverlaysInput and priceCrossBelowSma200, 'Price Cross Below SMA 200', 'Price crossed below SMA 200')
alertcondition(enableProOverlaysInput and bullishVolumeMomentum, 'Bullish Volume Momentum', 'Bullish volume momentum signal detected')
alertcondition(enableProOverlaysInput and bearishVolumeMomentum, 'Bearish Volume Momentum', 'Bearish volume momentum signal detected')
alertcondition(enableProOverlaysInput and ichimokuBullTkCross, 'Ichimoku Bullish TK Cross', 'Tenkan crossed above Kijun')
alertcondition(enableProOverlaysInput and ichimokuBearTkCross, 'Ichimoku Bearish TK Cross', 'Tenkan crossed below Kijun')

// Pro dashboard
var table proDashboard = table.new(position.top_right, 2, 11, border_width = 1)
if barstate.islast
    dashboardBg = color.new(#0f172a, 15)
    labelBg = color.new(#1e293b, 0)
    valueBg = color.new(#111827, 0)
    valueColor = color.white

    if showProDashboardInput
        table.cell(proDashboard, 0, 0, 'MODE', text_color = color.white, bgcolor = labelBg, text_size = size.small)
        table.cell(proDashboard, 1, 0, performanceModeInput, text_color = valueColor, bgcolor = valueBg, text_size = size.small)
        table.cell(proDashboard, 0, 1, 'PRESET', text_color = color.white, bgcolor = labelBg, text_size = size.small)
        table.cell(proDashboard, 1, 1, profilePresetInput, text_color = valueColor, bgcolor = valueBg, text_size = size.small)
        table.cell(proDashboard, 0, 2, 'SMC', text_color = color.white, bgcolor = labelBg, text_size = size.small)
        table.cell(proDashboard, 1, 2, enableSmartMoneyInput ? 'ON' : 'OFF', text_color = enableSmartMoneyInput ? color.lime : color.red, bgcolor = dashboardBg, text_size = size.small)
        table.cell(proDashboard, 0, 3, 'FIBFANS', text_color = color.white, bgcolor = labelBg, text_size = size.small)
        table.cell(proDashboard, 1, 3, enableFibFansInput ? 'ON' : 'OFF', text_color = enableFibFansInput ? color.lime : color.red, bgcolor = dashboardBg, text_size = size.small)
        table.cell(proDashboard, 0, 4, 'PRO', text_color = color.white, bgcolor = labelBg, text_size = size.small)
        table.cell(proDashboard, 1, 4, enableProOverlaysInput ? 'ON' : 'OFF', text_color = enableProOverlaysInput ? color.lime : color.red, bgcolor = dashboardBg, text_size = size.small)
        table.cell(proDashboard, 0, 5, 'SMA TF', text_color = color.white, bgcolor = labelBg, text_size = size.small)
        table.cell(proDashboard, 1, 5, smaTfFinal, text_color = valueColor, bgcolor = valueBg, text_size = size.small)
        table.cell(proDashboard, 0, 6, 'VOL TF', text_color = color.white, bgcolor = labelBg, text_size = size.small)
        table.cell(proDashboard, 1, 6, volumeMomentumTfFinal, text_color = valueColor, bgcolor = valueBg, text_size = size.small)
        table.cell(proDashboard, 0, 7, 'ICHI TF', text_color = color.white, bgcolor = labelBg, text_size = size.small)
        table.cell(proDashboard, 1, 7, ichimokuTfFinal, text_color = valueColor, bgcolor = valueBg, text_size = size.small)
        table.cell(proDashboard, 0, 8, 'VOL LEN', text_color = color.white, bgcolor = labelBg, text_size = size.small)
        table.cell(proDashboard, 1, 8, str.tostring(volumeMomentumLengthFinal), text_color = valueColor, bgcolor = valueBg, text_size = size.small)
        table.cell(proDashboard, 0, 9, 'VOL X', text_color = color.white, bgcolor = labelBg, text_size = size.small)
        table.cell(proDashboard, 1, 9, str.tostring(volumeThresholdFinal, '#.##'), text_color = valueColor, bgcolor = valueBg, text_size = size.small)
        table.cell(proDashboard, 0, 10, 'BIAS', text_color = color.white, bgcolor = labelBg, text_size = size.small)
        table.cell(proDashboard, 1, 10, close > sma200Value ? 'Above SMA200' : 'Below SMA200', text_color = close > sma200Value ? color.lime : color.red, bgcolor = dashboardBg, text_size = size.small)
    else
        for col = 0 to 1
            for row = 0 to 10
                table.cell(proDashboard, col, row, '', text_color = color.new(color.white, 100), bgcolor = color.new(color.black, 100), text_size = size.small)

var table logo = table.new(position.bottom_right, 1, 1)
if enableFibFansInput and barstate.islast
    table.cell(logo, 0, 0, '☼☾  ', text_size=size.normal, text_color=color.teal)
