# Display prettified date from input date field

Here is an exercise in 'How much JavaScript can you write to make something cool happen?'

*TL:DR*: If you know JavaScript, don't read the rest and just look at the code. There are comments in there enough to move you along.

## What is the problem?

I have a design where I need to allow a user to interact with a calendar in the view, select a date and then display that date within the view. The date needs to be prettified, e.g. I can't display `2016-12-25`, but I need to show `December 25th, 2016`.

There are a few other constraints in there as well. The main one is that I don't want the `<input>` to appear in the view on mobile. On desktop, for this example, I am leaving the `<input>` to allow users to update the date. In a more aggressive example I would replace the default HTML5 calendar with something more awesome ... but one thing at a time here.

But if you are interested in seeing this, [check this repo out](https://github.com/blackfalcon/dynamic-cal-pikaday-support).

## Solution; add JavaScript o_O

To set this up, we will need a few things. First we need some simple HTML to set up the DOM. Next we will need to add very little (for this example) CSS. Last we need to build out some JavaScript functions. This is, of course, the largest part of this. I will try to break this down into digestible chunks to make this easy to follow.

Here is a list of functions we need to build:

* get the current date, offset by time zone;
* get value from input and split individual date values to year, month, day;
* convert month value to string;
* read day and decipher ordinal expression (st, nd, rd, or th);
* return updated string of month, day and year and display in the view;
* manage events for clicking on displayed date to update/change displayed date

Phew ... that's a mouth full.

## The HTML

Like I said, pretty simple. For this example, we only need two DOM elements. Shown below, we need an `<input type="date" />` element with an id of `event-date` and a class of `hide`.

To display the value from the input element, we simply have a `div` with the id of `display-cal-value` and a class of `display-cal-value` as I don't like to use the same selector for functionality and presentation.

```
<input id="event-date" type="date" class="hide" value="" />
<div id="display-cal-value" class="display-cal-value"></div>
```

## The CSS

To keep the example really simple. Just adding some margin to the displayed calendar value and a class to hide the input.

It's important to keep in mind that this will not work with `display: none;`.

```
.display-cal-value {
    margin-top: 20px;
}

.hide {
    position: absolute;
    width: 1px;
    height: 1px;
    margin: -1px;
    padding: 0;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    border: 0;
}
```

## The JavaScript

In the introduction we had a healthy laundry list of things we need to build. First things first, we need to get today's date, based on timezone, and return this as a value that we can use.

### Current date value

To do this, we are going to build a function off of the `Date.prototype` object. `setDateInputValue` is the name of the function we will create. Within that function we need to set a variable for `localDate` by using the Date instance.

Using the value set to that variable, we need to use the `setMinutes` function to get a normalized value based on timezone and a 12 hour clock.

Last, we need this function to return a value that we can use.

The return is interesting because we need to mutate the value set by `localDate`. `localDate` by itself would return something like,

```
Wed Dec 21 2016 09:05:22 GMT-0800 (PST)
```

We can't use that. So by using the `toJSON()` function, we get,

```
2016-12-21T15:06:28.951Z
```

Again, not totally usable. This is where the `slice()` function helps to only return the first 10 digits. `2016-12-21` that's what we want.

```
Date.prototype.setDateInputValue = (function() {
    var localDate = new Date(this);
    localDate.setMinutes(this.getMinutes() - this.getTimezoneOffset());
    return localDate.toJSON().slice(0,10);
});
```
### Get and set current date to input

At this point we have a date input element with no value assigned to it. The next thing we need to do is create a function that will make use of the `setDateInputValue()` function we previously created.

Let's create a new function and call that `getSetCurrentDate`. In here we will do a few simple functions; find the input element and set the value to the current date as learned from the `setDateInputValue()` function.

We then need to create the `calendarInput` variable to that of the DOM element with the ID of `event-date`. Next, we use that variable and the `value` method to set the value of the element using the `new Date` prototype and the `setDateInputValue()` function.

```
function getSetCurrentDate () {
    var calendarInput = document.getElementById('event-date');

    calendarInput.value = new Date().setDateInputValue();
};
```

At this point you should have a input element that on refresh will have the current date in it. But what can we do with this new value?

In that same function we need to add another variable that finds the DOM element where we want the date to show up in the view.

```
var displayCalValue = document.getElementById('display-cal-value');
```

Then we can set the value to that variable and the `innerHTML` method to that of the value of `calendarInput.value`.

```
displayCalValue.innerHTML = calendarInput.value
```

With this so far, you should have something like `2016-12-21` appear in the browser.

### Ordinal numbers

Now that we have the current date and it's being displayed to the browser, we need to clean this up so that it's more presentable.

#### Change number into month string

First things first, how do we convert `12` to `December`? There are a few ways to do this, but one that is pretty straight forward is that we have a bunch of `if` statements. Basically, `if month == 12 then December`, that will work, but there is a better way.

We can use a switch statement inside a function, we can define a series of cases and the value returned based on the case.

Let's create a new function called `utcMonthString`. This function will take a single argument, an integer, we'll name it `i`.

Last, to use this function, we need to return the final value for a month.

```
function utcMonthString(i) {
    // set empty variable
    var month;

    // use switch cases to evaluate the value of i
    switch(i) {
        case 1: month="January"; break;
        case 2: month="February"; break;
        case 3: month="March"; break;
        case 4: month="April"; break;
        case 5: month="May"; break;
        case 6: month="June"; break;
        case 7: month="July"; break;
        case 8: month="August"; break;
        case 9: month="September"; break;
        case 10: month="October"; break;
        case 11: month="November"; break;
        case 12: month="December"; break;
    }

    // if i == 3, then month == March
    return month;
}
```

#### Set ordinal suffix

With that part done, we need to address another part of the prettified date, the ordinal suffix (st, nd, rd and th). Let's create a new function called `ordinalSuffix()` and similar to the previous function, we have a single argument for an integer, `i`.

This one gets fun as we are doing some [modulo math](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Remainder) to find the remainder of an integer as shown below.

Dividing the value of `i` by `10` and `100` we will get the modulo of that value and assign to the appropriate variables.

Using a series of `if` statements and determining that the value does not == `11`, `12`, or `13`, then we can determine the ordinal suffix added to the number.

```
function ordinalSuffix(i) {
    var modulo10 = i % 10,
        modulo100 = i % 100;
    if (modulo10 == 1 && modulo100 != 11) {
        return i + "st";
    }
    if (modulo10 == 2 && modulo100 != 12) {
        return i + "nd";
    }
    if (modulo10 == 3 && modulo100 != 13) {
        return i + "rd";
    }
    return i + "th";
}
```

#### Parse date number

Last, we need to make one more function that will parse that date value as I mentioned before, assign the parsed values to named variables, make use of the `ordinalSuffix()` and `utcMonthString()` functions, then return a string that we can display in the browser.

Create the function `displayOrdinalDate()` that takes the argument of `date` that we will pass in at the time of use.

Inside that function we need to split that date value by the character `-` and assign that to the variable of `dateArray`. This will return an array something like,

```
["2016", "12", "21"]
```

Now split, we can then create values for the `year`, `month`, and `day` variables based on it's place in the array.

But notice how the values in the array have `" "` around them? That means they are now strings, not numbers.

Splitting the number converts it to a `string`, so we need to change the `string` back to a `number`. This is done using the `Number()` function, as you see below.

Create two new variables, `ordinalDay` and `displayMonth`. `ordinalDay` uses the `ordinalSuffix()` function we created and passes in the variable for `day` inside the `Number()` function and returns the value of `i` plus the correct suffix.

Then `displayMonth` uses the `utcMonthString()` function we created, and uses the `month` variable, again inside the `Number()` function.

The final part of this is to return a string that concatenates all this new data and get back something that looks like `December 21st, 2016`.

```
function displayOrdinalDate(date) {
    var dateArray = date.split('-'),
        year = dateArray[0],
        month = dateArray[1],
        day = dateArray[2],

        // updates date value to that of value with ordinal indicator
        ordinalDay = ordinalSuffix(Number(day)),

        // updates month value to that of month name
        displayMonth = utcMonthString(Number(month));

    return displayMonth + " " + ordinalDay + "," + " " + year;
}
```

At this point you will have a input that gets the current date, that date is run through some processes to return a prettified value. That updated value is then presented to the browser view.

Pretty cool, but what about updating the date?

### Update the view with new date

To start wrapping up this experience, we need the view to update every time a user changes the date in the input element.

To do this we will create another function called `updateDisplayDateEvent`. In this function we need to set two new variables, `calendarInput` and
`displayCalValue`. You may remember these variables from a previous function, but since those variables are not global, we need to reset them.

Next thing we need is an event trigger that will change the DOM to reflect the value of the input element. For this we will enclose another function that will be triggered by the `onchange` event on `calendarInput`.

Inside this scope we need new values for `calendarInputDate` which will be that of `calendarInput.value`. Using `innerHTML` we will update the DOM content of `displayCalValue` to that of `calendarInputDate` using the `displayOrdinalDate()` function.

```
// change event that updates display date based on input
function updateDisplayDateEvent() {
    var calendarInput = document.getElementById('event-date');

    // change event to set new value to DOM;
    calendarInput.onchange = function() {
      var calendarInputDate = calendarInput.value,
          defaultValue = new Date().setDateInputValue();

      displayCalValue.innerHTML = displayOrdinalDate(calendarInputDate);
    }
};
```

At this point you will have a UI with the current date, and when that date field is updated, this will automatically update the value of the prettified string in the view.

What if the user clears the input data? This will return a `NaN`, not a number. We can guard ourself against this, and if the user clears the date, we then can re-instate the current date using all the tools we already created wrapped in an `if` statement. We also need to reset the `displayCalValue` variable too so that we know where to write the data back to.

```
var displayCalValue = document.getElementById('display-cal-value');
```

We can use the native function of `isNaN()` to evaluate the number we are getting back from the input element. This function returns a boolean value that the `if` statement will evaluate.

So, if the value comes back as a number, then apply `displayOrdinalDate(calendarInputDate)` to `displayCalValue`.

If it is not a number, then we need a new variable we will call `defaultValue`. This will simply set a new date object, `new Date().setDateInputValue()`. We need to add this variable inside the `calendarInput.onchange` function.

```
defaultValue = new Date().setDateInputValue();
```

Last, we leverage the `displayCalValue` DOM object, apply the `innerHTML` method and again use the `displayOrdinalDate()` function with the argument of our new `defaultValue` value.

```
if(isNaN(calendarInputDate)) {
    displayCalValue.innerHTML = displayOrdinalDate(calendarInputDate);
} else {
    calendarInput.value = defaultValue;
    displayCalValue.innerHTML = displayOrdinalDate(defaultValue);
}
```

Doing this, the user will never see the generic `mm/dd/yyyy` value of the input. Clearing the date will always return today's date.

## Turn this experience up to 11

At this time, we pretty much have things wired up really well. The input gets the current date and we can transform that date into a prettified string and present that back to the view.

Can we update this so that when the user clicks in the prettified date, this will focus on the input? And what about hiding the input until requested? And what about a mobile experience? All great questions.

Remember the `hide` CSS we added? Well, lets create a function that removes that selector when we click on the prettified string thus exposing the input and then hides it when we click away.

To do this, much like the `onchange` even we created before, we are going to create new events, `onblur` and `onclick`.

First we will address the blur event. This simply is JavaScript's way fo saying that the user no longer has focus on that element. This function us pretty simple, since we are inside the `updateDisplayDateEvent()` function, we still have access to the `calendarInput` value. Couple this with the `className` method on the element, we can set the CSS to use the class of `hide`.

```
calendarInput.onblur = function() {
    calendarInput.className = "hide";
}
```

Next we want to trigger the exposing of the input when we click on it, using the `onclick` event trigger. But this has a caveat, we only want this to show up on desktop, not mobile. But how?

In JavaScript there is the `matchMedia` function that works much like `@media` in CSS. We can pass in a `min-width` and this will control the event that fires in the browser. What this function is saying is, if the browser has a min-width of `769px` then remove the class name of `hide` from `calendarInput` and set focus. If the browser does not meet the if statement, then only set focus.

What does this do? Regardless of browser we want to set focus on the element. But only on desktop do we want to see the input. On mobile, when you set focus the device will show their native date selector. This works out really well for our purposes.

```
displayCalValue.onclick = function() {
    var mq = window.matchMedia('(min-width: 769px)');

    if(mq.matches) {
        calendarInput.className = "";
    };
    calendarInput.focus();
}
```

## In conclusion

That was a mouth full. This tutorial touched on so many JavaScript ideas. Getting current data, setting value, converting data into another value, displaying new value and allowing for multiple interactions.

If you are new to JavaScript, I hope that you learned something. If you are experienced in JavaScript, I'd love some feedback if you see a way to make this even better.

In this repo is a final HTML document that has all the code. Feel free to open in a browser and play with the codes.
