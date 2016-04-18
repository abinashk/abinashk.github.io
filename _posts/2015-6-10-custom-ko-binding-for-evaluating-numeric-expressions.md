---
layout: post
title: Allowing simple arithmetic expression in numeric field using custom KO binding 
excerpt: "Using eval() function we can provide a custom form field where user can input arithmetic expression which will get evaluated on focus out"
tags: [knockoutjs, custom-binding, numeric]
modified: 2015-6-10
comments: true
---
A custom KO Binding for evaluating simple expressions in place of plain numeric form field.

{% highlight javascript linenos=table %}

ko.bindingHandlers.eval = {
    // Default init handler
    init: function (element, valueAccessor) {
    },
    // Update handler, this is the interesting part
    update: function (element, valueAccessor) {
        
        // Access value in normal text field
        var value = ko.utils.unwrapObservable(valueAccessor());
        var value_initial = value;
        if (typeof value == 'undefined')
            return;
        
        // If not undefined
        try {

            if (typeof value.indexOf == 'function' && value.indexOf('%') > 0)
                // handle % before anything else because eval() doesn't handle it
                value = calculate_percent(value);
            
            // using eval() method to compute the expression
            var val = eval(value);

            // if number greater than exp 10 then change css to invalid-cell
            if (val != '' && val != null && typeof val != 'undefined'){
            var exponent = Number(val.toExponential().split('e')[1]);
            }
            if(exponent>10 ){
                $(element).addClass('invalid-cell');
                val = value_initial.toString()+' -too large';
                var observable = valueAccessor();
                observable(val);
                $(element).text(val);
            }

            // if number is negative then change css to invalid-cell
            else if(val < 0){
                $(element).addClass('invalid-cell');
                val = value_initial.toString()+' -ve number';
                var observable = valueAccessor();
                observable(val);
                $(element).text(val);
            }

            // else replace the input expression by the computed value
            else{
                temp_val =  parseFloat(val);
                val = temp_val.toString();
                $(element).text(val);
                var observable = valueAccessor();
                observable(val);
                $(element).removeClass('invalid-cell');
            }

        } catch (e) {
            $(element).addClass('invalid-cell');
        }
    }
}

// method to handle %
calculate_percent = function (str) {
    str = str.toString();
    str = str.replace(/ /g, '');
    str = str.replace(/([0-9]+)([\+\-\*\/]{1})([0-9]+)%/, function (s, n1, o, n2) {
        var n1 = parseFloat(n1);
        var n2 = parseFloat(n2);
        if (o == '+') {
            return n1 + n1 * n2 / 100;
        }
        if (o == '-') {
            return n1 - n1 * n2 / 100;
        }
        if (o == '*') {
            return n1 * n2 / 100;
        }
        if (o == '/') {
            return 100 * n1 / n2;
        }
        return s;
    });
    return str;
}

{% endhighlight %}