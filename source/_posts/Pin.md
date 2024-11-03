---
seo_title: Pin
comment: false
layout: page
robots: noindex,nofollow
readmore: false
archive: false
---
<style>
:root {
  --color-calendar-graph-day-bg: #dddbdb;
  --color-calendar-graph-day-L1-bg: #39dd34;
  --color-calendar-graph-day-L2-bg: #45a045;
  --color-calendar-graph-day-L3-bg: #047526;
  --color-calendar-graph-day-L4-bg: #0a4208;
}
rect.ContributionCalendar-day[data-level='0'] {
    fill: var(--color-calendar-graph-day-bg);
}
rect.ContributionCalendar-day[data-level='1'] {
    fill: var(--color-calendar-graph-day-L1-bg);
}
rect.ContributionCalendar-day[data-level='2'] {
    fill: var(--color-calendar-graph-day-L2-bg);
}
rect.ContributionCalendar-day[data-level='3'] {
    fill: var(--color-calendar-graph-day-L3-bg);
}
rect.ContributionCalendar-day[data-level='4'] {
    fill: var(--color-calendar-graph-day-L4-bg);
}
.calendar .width-full > .float-left {
    display: none;
}
.calendar {
    min-height: 243px;
    text-align: center;
    margin: 0 auto;
}
.calendar-graph text.wday,
.calendar-graph text.month {
    font-size: 10px;
    fill: #aaa;
}
.contrib-legend {
    text-align: right;
    padding: 0 14px 10px 0;
    display: inline-block;
    float: right;
}
.contrib-legend .legend {
    display: inline-block;
    list-style: none;
    margin: 0 5px;
    position: relative;
    bottom: -1px;
    padding: 0;
}
.contrib-legend .legend li {
    display: inline-block;
    width: 10px;
    height: 10px;
}
.text-small {
    font-size: 12px;
    color: #767676;
}
.calendar-graph {
    padding: 5px 0 0;
    text-align: center;
}
.contrib-column {
    padding: 15px 0;
    text-align: center;
    font-size: 11px;
}
.contrib-column-first {
    border-left: 0;
}
.table-column {
    display: table-cell;
    width: 1%;
    padding-right: 10px;
    padding-left: 10px;
    vertical-align: top;
}
.contrib-number {
    font-weight: 300;
    line-height: 1.3em;
    font-size: 24px;
    display: block;
    color: #333;
}
.calendar img.spinner {
    width: 70px;
    margin-top: 50px;
    min-height: 70px;
}
.monospace {
    text-align: center;
    color: #000;
}
.monospace a {
    color: #1D75AB;
    text-decoration: none;
}
.contrib-footer {
    font-size: 11px;
    padding: 0 10px 12px;
    text-align: left;
    width: 100%;
    box-sizing: border-box;
    height: 26px;
}
.left.text-muted {
    float: left;
    margin-left: 9px;
    color: #767676;
}
.left.text-muted a {
    color: #4078c0;
    text-decoration: none;
}
.left.text-muted a:hover,
.monospace a:hover {
    text-decoration: underline;
}
h2.f4.text-normal.mb-3 {
    display: none;
}
.float-left.text-gray {
    float: left;
}
#user-activity-overview{
    display:none;
}
.day-tooltip {
    white-space: nowrap;
    position: absolute;
    z-index: 99999;
    padding: 10px;
    font-size: 12px;
    color: #959da5;
    text-align: center;
    background: rgba(0,0,0,.85);
    border-radius: 3px;
    display: none;
    pointer-events: none;
}
.day-tooltip strong {
    color: #dfe2e5;
}
.day-tooltip.is-visible {
    display: block;
}
.day-tooltip:after {
    position: absolute;
    bottom: -10px;
    left: 50%;
    width: 5px;
    height: 5px;
    box-sizing: border-box;
    margin: 0 0 0 -5px;
    content: " ";
    border: 5px solid transparent;
    border-top-color: rgba(0,0,0,.85)
}
text.ContributionCalendar-label {
    fill: #ccc;
    font-size: 11px;
}
</style>
<script src="https://unpkg.com/github-calendar@latest/dist/github-calendar.min.js"></script>
<div class="calendar">Loading the data just for you.</div>
<script data-pjax>
    GitHubCalendar(".calendar", "SukiEva", { responsive: true });
</script>
<!-- more -->