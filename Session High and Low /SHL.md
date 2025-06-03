//@version=5
indicator("Session High/Low + Daily Open - NY Time", overlay=true, max_lines_count=500)

// === INPUTS ===
// Session High/Low Colors
asiaHighColor = input.color(color.gray, "Asia High Color")
asiaLowColor = input.color(color.gray, "Asia Low Color")
londonHighColor = input.color(color.gray, "London High Color")
londonLowColor = input.color(color.gray, "London Low Color")
nyHighColor = input.color(color.gray, "NY High Color")
nyLowColor = input.color(color.gray, "NY Low Color")
lineW = input.int(1, "Line Width")

// Daily Open Line Inputs
showDailyOpen = input.bool(true, "Show Daily Candle Open Line")
showAllDailyOpens = input.bool(true, "◽ Show All Daily Opens", tooltip="When unchecked, only shows current day's open")
dailyOpenColor = input.color(color.white, "Daily Open Line Color")

// === TIME ZONE OFFSET ===
nyOffset = -4 // UTC-4 for NY Daylight Time
timeNY = time + nyOffset \* 3600000
hourNY = hour(timeNY)
dayChange = ta.change(time("D"))

// === SESSION HOURS in NY TIME ===
inAsia = (hourNY >= 18 or hourNY < 2)
inLondon = (hourNY >= 3 and hourNY < 9)
inNY = (hourNY >= 9 and hourNY < 17)

// === VARS ===
// Session High/Low Variables
var float asiaHigh = na
var float asiaLow = na
var float londonHigh = na
var float londonLow = na
var float nyHigh = na
var float nyLow = na

var int asiaHighBar = na
var int asiaLowBar = na
var int londonHighBar = na
var int londonLowBar = na
var int nyHighBar = na
var int nyLowBar = na

// Daily Open Line Variables
var float dailyOpen = na
var int dailyOpenBar = na
var line[] dailyLines = array.new_line()
var line currentDayLine = na

// === DAILY OPEN LOGIC ===
if dayChange
// Reset Session High/Lows
asiaHigh := na
asiaLow := na
londonHigh := na
londonLow := na
nyHigh := na
nyLow := na

    asiaHighBar := na
    asiaLowBar := na
    londonHighBar := na
    londonLowBar := na
    nyHighBar := na
    nyLowBar := na

    // Set Daily Open
    dailyOpen := open
    dailyOpenBar := bar_index

    // Clean up current day line if it exists
    if not na(currentDayLine)
        line.delete(currentDayLine)
        currentDayLine := na

    if showDailyOpen and showAllDailyOpens
        // For all days mode - add to array
        barsPerDay = math.max(1, 1440 / timeframe.multiplier)
        lineEnd = bar_index + math.min(barsPerDay * 2, 500)
        l = line.new(bar_index, dailyOpen, lineEnd, dailyOpen, color=dailyOpenColor, extend=extend.none)
        array.push(dailyLines, l)

// For current day only mode - draw/update line
if showDailyOpen and not showAllDailyOpens and not na(dailyOpen)
if na(currentDayLine)
currentDayLine := line.new(dailyOpenBar, dailyOpen, dailyOpenBar + 1, dailyOpen, color=dailyOpenColor, width=1, extend=extend.right)
else
line.set_xy1(currentDayLine, dailyOpenBar, dailyOpen)
line.set_xy2(currentDayLine, dailyOpenBar + 1, dailyOpen)

// === CLEAN UP OLD DAILY LINES ===
if showAllDailyOpens and array.size(dailyLines) > 500
line.delete(array.shift(dailyLines))

// === Track Session Highs and Lows ===
if inAsia
if na(asiaHigh) or high > asiaHigh
asiaHigh := high
asiaHighBar := bar_index
if na(asiaLow) or low < asiaLow
asiaLow := low
asiaLowBar := bar_index

if inLondon
if na(londonHigh) or high > londonHigh
londonHigh := high
londonHighBar := bar_index
if na(londonLow) or low < londonLow
londonLow := low
londonLowBar := bar_index

if inNY
if na(nyHigh) or high > nyHigh
nyHigh := high
nyHighBar := bar_index
if na(nyLow) or low < nyLow
nyLow := low
nyLowBar := bar_index

// === Session end logic ===
asiaEnd = hourNY == 2 and hourNY[1] != 2
londonEnd = hourNY == 9 and hourNY[1] != 9
nyEnd = hourNY == 17 and hourNY[1] != 17

// === Draw Session High/Low Lines ===
var line asiaHLine = na
var line asiaLLine = na
var line londonHLine = na
var line londonLLine = na
var line nyHLine = na
var line nyLLine = na

if asiaEnd and not na(asiaHigh)
line.delete(asiaHLine)
line.delete(asiaLLine)
asiaHLine := line.new(asiaHighBar, asiaHigh, bar_index + 50, asiaHigh, extend=extend.right, color=asiaHighColor, width=lineW)
asiaLLine := line.new(asiaLowBar, asiaLow, bar_index + 50, asiaLow, extend=extend.right, color=asiaLowColor, width=lineW)

if londonEnd and not na(londonHigh)
line.delete(londonHLine)
line.delete(londonLLine)
londonHLine := line.new(londonHighBar, londonHigh, bar_index + 50, londonHigh, extend=extend.right, color=londonHighColor, width=lineW)
londonLLine := line.new(londonLowBar, londonLow, bar_index + 50, londonLow, extend=extend.right, color=londonLowColor, width=lineW)

if nyEnd and not na(nyHigh)
line.delete(nyHLine)
line.delete(nyLLine)
nyHLine := line.new(nyHighBar, nyHigh, bar_index + 50, nyHigh, extend=extend.right, color=nyHighColor, width=lineW)
nyLLine := line.new(nyLowBar, nyLow, bar_index + 50, nyLow, extend=extend.right, color=nyLowColor, width=lineW)
