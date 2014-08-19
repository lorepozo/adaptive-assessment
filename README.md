#The Adaptive Assessment Module
##(a.k.a. Problem Bombarder)

***

### Getting Started

You can see an example implementation with the provided problems.json file [here](http://lucasmorales.co/resources/relate/assess.html?js=problems).

To open the implementation yourself, download the reposity and use your web browser to navigate:

> ...assess.html?js=problems

This will load the problems.json file inside assess.html. 

### Using with edX

This module was written with edX's JSInput XBlock in mind. To use this tool with edX, simply use the following XML snippet, changing `width`, `height`, and `problems` accordingly.

	<script type="loncapa/python">
	import json
	def assess(e, a):
	  return json.loads(a)["answer"]
	</script>
	<customreponse cfn="assess">
    	<jsinput gradefn="JSInput.getGrade"
    		get_statefn="JSInput.getState"
    		set_statefn="JSInput.setState"
    		width="600"
    		height="320"
    		html_file="/static/assess.html?js=problems" />
	</customresponse>

### The JSON Format

    ProblemName.json
	
+ `"problems"`: __Array__ of problem objects - see __The Problem Object__ below
+ `"method"`: __String__(/__Object__) _optional_
	+ `"norepeatrandom"` (default) problems occur in random order
	+ `"repeatrandom"` problems incorrectly answered may reappear, occur in random order
	+ `"order"` follows order of `problems` array
	+ `"bisection"` uses problem difficulty to hone on student skill
	+ `"random"` entirely random, _complete_ repetition possible
	+ `{"Function":"CODE"}` returns problem __Object__ - see __Custom Functions__ below
+ `"grade"`: __String__(/__Object__) _optional_
	+ `"mix"` (default) -> same as mix60%3 (pass with either 60% or 3 consecutive correct)
	+ NOTE: Below, `PP` must be two digits and `N` must be at least one digit
	+ `"mixPP%N"` pass with greater than or equal to PP% correct, or with N consecutive correct
	+ `"PP%"` pass with greater than or equal to PP% correct
	+ `"N"` pass with N consecutive correct answers
	+ `{"Function":"CODE"}` returns __Boolean__ - see __Custom Functions__ below
+ `"feedback"`: __Array__ of feedbacks, of which the most complex will be shown if matched
	+ __Array__ for a single feedback (e.g. `[[0,1],"Comeback time!"]`)
		+ __Array__ of `1`s and `0`s for correct/incorrect matching - longer/most complex match takes precedence
		+ __String__ of the actual feedback (Text/Html/MathJax)
+ `"explain"`: __Boolean__ _option_ forces "View Solution" button capability (see `explanation` in __The Problem Object__)

### The Problem Object

A _problem object_ conclusively represents an individual problem in the form of JSON.

Each problem object has the following keys:

+ `"id"`: __String__ A unique identifier for each problem
+ `"difficulty"`: __Number__ _optional_ As in Item Response Theory, measured in standard deviations (0 is average and default)
+ `"weight"`: __Number__ _optional_  A scaling factor for percentage-based grading (see `grade` in __The JSON Format__), defaults to 1
+ `"type"`: __String__
	+ `"text"`
	+ `"checkbox"`
	+ `"multiplechoice"`
	+ `"dropdown"`
+ `"content"`:
	+ _text_ 
		+ __String__ The question/prompt (Text/Html/MathJax)
	+ _checkbox/multiplechoice/dropdown_
		+ __Object__
			+ `"question"`: __String__ The question/prompt (Text/Html/MathJax)
			+ `"options"`: __Object__ displayed lexicographically according to key
				+ `"key":"Text/Html/MathJax"` repeat as needed
+ `"solution"`:
	+ _text_
		 + __String__ For exact string match
		 + __Object__ `{"Function":"CODE"}` returns __Boolean__ - see __Custom Functions__ below
	+ _checkbox_
		+ __Array__ of the keys corresponding to correct options
	+ _multiplchoice/dropdown_
		+ __String__ of the key corresponding to the correct option
+ `"explanation"`: __String__ Shown if using a non-repeating `method` or if `explain` is set to true (Text/Html/MathJax)

### Custom Functions

special available values/functions:

+ `problemFromId`: __Function__ takes a problem ID, returns the corresponding problem object or `null`
+ `problems`: __Array__ Parsed from json
+ `exp`: __Array__ Problem IDs in order experienced
+ `g`: __Array__ Booleans representing correctness in order experienced

If the custom function is for a problem object's solution, an additional value of `inp` is provided, and is a __String__ representing the user's input. An example for a numerical text solution would be `"solution":{"Function":"return Number(inp)==0"}`

Below is an example that will make each new problem be the same as the last incorrect problem or (if none are incorrect, the last problem defined in `problems`): 

__json___:

```JSON  
"method": {
    "Function": "for(var i=0;i&lt;g.length;i++)if(!g[g.length-1-i])return problemFromId(exp[exp.length-1-i]);return problems[problems.length-1];"
} 
```
	
beautified __javascript__ used above:

```Javascript
// iterate over assessed problems
for (var i = 0; i < g.length; i++) {
	// note that exp.length == g.length
	// check if incorrect in reverse order
	if ( !g[g.length-1 - i] ) {
		// return incorrectly answered problem
		return problemFromId( exp[exp.length-1 - i] );
	}
}
// if nothing comes from the code above, return that last defined problem
return problems[problems.length-1];
```
