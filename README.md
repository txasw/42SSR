# 42 Intra Slot Calendar Reload

A tiny JavaScript utility to **soft-reload the 42 Intra project slot calendar** without refreshing the entire page.  
It re-initializes the calendar on the fly and fetches fresh slot data directly from the page's `data-index-url` endpoint.

---

## 💡 Why This Exists

At 42 Experience, getting a project evaluation slot can be competitive, especially when many students are trying to book at the same time.
Refreshing the whole page takes extra time and can even make you lose the slot to someone else.  

This script was created to:

- Quickly check for the **nearest available evaluation slot**
- Reload the slot data **without reloading the entire page**
- Make the experience smoother and faster, especially when slots are limited

---

## ✨ Features

- One-click reload of the calendar data
- Uses the page’s own API (`data-index-url`) to fetch slots
- No page refresh needed, keeping your place on the page
- Works seamlessly with the original 42 calendar UI (uses FullCalendar’s API)
- Optional floating "Reload Slots" button for easy access
- Can be customized to auto-refresh every X seconds if needed

---

## 🛠 How to Use

1. Open your project slot page on [projects.intra.42.fr](https://projects.intra.42.fr)
2. Click **Subscribe to defense** to open the slot calendar
3. Open **DevTools → Console**
4. Paste one of the scripts below
5. Press `Enter`

---

### 🔄 Manual Reload (Console Only)

Use this if you just want to reload slots once

```js
(function(){
  var $ = window.jQuery;
  if(!$ || !$.fn.fullCalendar) return console.error('FullCalendar not found');
  var $cal = $('#calendar');
  if(!$cal.length) $cal = $("<div id='calendar'></div>").prependTo($('.container-item').length?'.container-item':'body');
  try{ if($cal.data('fullCalendar')) $cal.fullCalendar('destroy'); }catch(e){}
  var indexUrl=$cal.data('index-url')||$cal.attr('data-index-url'), duration=parseInt($cal.data('duration')||1,10)||1, $modal=$('#slotSelectionModalContainer');
  function populateModal(ids,event){
    var start=moment(event._start||event.start);
    var $list=$modal.find('select').empty(),nb=Math.max(0,ids.length-duration+1);
    for(var i=0;i<nb;i++){var to=start.clone().add(15*duration,'m');$list.append($('<option>').val(i).text('From '+start.format('LT')+' to '+to.format('LT')));start.subtract((duration-1)*15,'m');}
    $list.data('ids',ids.join(','));
  }
  $cal.fullCalendar({
    header:{left:'',center:'title',right:'today prev next'},
    axisFormat:'h:mm a',
    defaultDate:moment(),
    defaultView:'agendaWeek',
    firstDay:1,
    allDaySlot:false,
    slotEventOverlap:false,
    slotDuration:'00:30:00',
    snapDuration:'00:15:00',
    events:(s,e,tz,cb)=>$.getJSON(indexUrl,{start:s.format(),end:e.format()}).done(cb).fail(()=>cb([])),
    eventClick:e=>{var ids=typeof e.ids==='string'?e.ids.split(','):(e.ids||(e.id?[e.id]:[]));populateModal(ids,e);$modal.modal('show');},
    loading:b=>$('#loading').toggle(b)
  });
  $cal.fullCalendar('refetchEvents');
})();
```

### 🖱 Version with Floating Button

This version adds a floating Reload Slots button at the top right corner of the page

```js
(function(){
  var $ = window.jQuery;
  if(!$ || !$.fn.fullCalendar) return console.error('FullCalendar not found');
  var $cal = $('#calendar');
  if(!$cal.length) $cal = $("<div id='calendar'></div>").prependTo($('.container-item').length?'.container-item':'body');
  try{ if($cal.data('fullCalendar')) $cal.fullCalendar('destroy'); }catch(e){}
  var indexUrl=$cal.data('index-url')||$cal.attr('data-index-url'), duration=parseInt($cal.data('duration')||1,10)||1, $modal=$('#slotSelectionModalContainer');
  function populateModal(ids,event){
    var start=moment(event._start||event.start);
    var $list=$modal.find('select').empty(),nb=Math.max(0,ids.length-duration+1);
    for(var i=0;i<nb;i++){var to=start.clone().add(15*duration,'m');$list.append($('<option>').val(i).text('From '+start.format('LT')+' to '+to.format('LT')));start.subtract((duration-1)*15,'m');}
    $list.data('ids',ids.join(','));
  }
  $cal.fullCalendar({
    header:{left:'',center:'title',right:'today prev next'},
    axisFormat:'h:mm a',
    defaultDate:moment(),
    defaultView:'agendaWeek',
    firstDay:1,
    allDaySlot:false,
    slotEventOverlap:false,
    slotDuration:'00:30:00',
    snapDuration:'00:15:00',
    events:(s,e,tz,cb)=>$.getJSON(indexUrl,{start:s.format(),end:e.format()}).done(cb).fail(()=>cb([])),
    eventClick:e=>{var ids=typeof e.ids==='string'?e.ids.split(','):(e.ids||(e.id?[e.id]:[]));populateModal(ids,e);$modal.modal('show');},
    loading:b=>$('#loading').toggle(b)
  });
  $cal.fullCalendar('refetchEvents');
  var $btn=$('<button>Reload Slots</button>').css({position:'fixed',top:'10px',right:'10px',zIndex:9999,padding:'6px 12px',border:'none',borderRadius:'6px',background:'#4CAF50',color:'#fff',cursor:'pointer'});
  $btn.on('click',()=> $cal.fullCalendar('refetchEvents'));
  $('body').append($btn);
})();
```

### 🚀 Optional: Auto-Reload Every X Seconds

If you want the page to auto-refresh slots every 30 seconds:

```js
setInterval(() => $('#calendar').fullCalendar('refetchEvents'), 30000);
```

### 📸 Screenshot
![capture](https://github.com/txasw/42SSR/blob/main/capture.gif)