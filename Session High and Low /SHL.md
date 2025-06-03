//@version=5
indicator("Session High/Low (Asia, London, NY) - NY Time - Exact Candle Alignment", overlay=true)

// === INPUTS ===
asiaHighColor = input.color(color.gray, "Asia High Color")
asiaLowColor = input.color(color.gray, "Asia Low Color")
londonHighColor = input.color(color.gray, "London High Color")
londonLowColor = input.color(color.gray, "London Low Color")
nyHighColor = input.color(color.gray, "NY High Color")
nyLowColor = input.color(color.gray, "NY Low Color")
lineW = input.int(1, "Line Width")

// === TIME ZONE OFFSET (New York Standard Time = UTC-5, Daylight = UTC-4)
// For simplicity, we assume Daylight Saving is active (UTC-4)
nyOffset = -4 // Adjust to -5 if DST is off
timeNY = time + nyOffset \* 3600000
hourNY = hour(timeNY)
dayChange = ta.change(time("D"))

// === SESSION HOURS in NY TIME ===
inAsia = (hourNY >= 18 or hourNY < 2)
inLondon = (hourNY >= 3 and hourNY < 9)
inNY = (hourNY >= 9 and hourNY < 17)

// === VARS ===
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

if dayChange
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

// === Track highs and lows ===
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

// === Draw horizontal lines ===
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
