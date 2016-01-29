---
layout: post
title: ReactJs & Webpack Notes
description: "This is a notes for ReactJs"
modified: 2016-01-20
tags: [react, webpack]
---
### React

#### Properties in JSX
- Label -> "for"

  Error message: "Warning: Unknown DOM property for. Did you mean htmlFor?".
  Use htmlFor replace "for".
- ReactDOM.render in document.body

  Error message: "Warning: render(): Rendering components directly into document.body is discouraged, since its children are often manipulated by third-party scripts and browser extensions. This may lead to subtle reconciliation issues. Try rendering into a container element created for your app."
  Creat a new div like: `ReactDOM.render(<App />, document.getElementById("root"));`

### Webpack
- Inline image

{% highlight javascript %}
import url from "../img/temp.png"
or
var url = require("../img/temp.png");

<img src={url} />
{% endhighlight %}
- How to debug orignal file

  add the following code into webpack.config.js
{% highlight javascript %}
module.exports = {
    entry: './js/entry.js',
    devtool: "source-map",
    ...
}
{% endhighlight %}
