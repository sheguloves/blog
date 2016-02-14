---
layout: post
title: Integrate front package tool into maven
description: "Integrate front package tool into maven"
modified: 2016-02-14
tags: [maven, webpack, node.js, npm]
---

### Background
正如我们所知，Maven无疑是java代码打包的首选（是不是唯一不清楚）工具，然而，对于前端而言，涌现出太多太多的打包工具，例如最新：grunt, gulp, webpack 等等。如何将前端javascript，html与后端一起打包呢，这就是我今天要写到的一个Maven Plugin：[Frontend maven plugin](https://github.com/eirslett/frontend-maven-plugin)

<!--more-->

### Implementation
在它的 [Github](https://github.com/eirslett/frontend-maven-plugin)上有比较清楚的文档，我在下面贴出我的pom文件，仅供参考。

pom.xml
{% highlight xml %}
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>**-parent</artifactId>
        <groupId>**id</groupId>
        <version>1.0.0</version>
    </parent>
    <artifactId>**</artifactId>
    <packaging>**</packaging>
    <properties>
        <node.version>v5.4.0</node.version>
        <npm.version>3.3.12</npm.version>
        <frontend.version>0.0.27</frontend.version>
    </properties>
    <build>
        <plugins>
            <plugin>
                <groupId>com.github.eirslett</groupId>
                <artifactId>frontend-maven-plugin</artifactId>
                <version>${frontend.version}</version>
                <configuration>
                    <workingDirectory>webapp</workingDirectory>
                </configuration>
                <executions>
                    <execution>
                        <id>install node and npm</id>
                        <goals>
                            <goal>install-node-and-npm</goal>
                        </goals>
                        <configuration>
                            <nodeVersion>${node.version}</nodeVersion>
                            <npmVersion>${npm.version}</npmVersion>
                        </configuration>
                    </execution>
                    <execution>
                        <id>npm install</id>
                        <goals>
                            <goal>npm</goal>
                        </goals>
                        <configuration>
                            <arguments>install</arguments>
                        </configuration>
                    </execution>
                    <execution>
                        <id>npm run build</id>
                        <goals>
                            <goal>npm</goal>
                        </goals>
                        <configuration>
                            <arguments>run build</arguments>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
{% endhighlight %}

下面是我的package.json文件。
{% highlight json %}
{
    "name": "monitor",
    "version": "3.0.0",
    "description": "This is a Neulion monitor web project",
    "main": "index.js",
    "scripts": {
        "build": "webpack --config webpack.config.js --progress --colors"
    },
    "keywords": [
        "monitor",
        "Neulion",
        "react",
        "reactjs",
        "hot",
        "webpack"
    ],
    "author": "Guofeng Xi",
    "license": "ISC",
    "dependencies": {
        "date-format": "0.0.2",
        "jquery": "^2.2.0",
        "react": "^0.14.6",
        "react-addons-css-transition-group": "^0.14.6",
        "react-dom": "^0.14.6"
    },
    "devDependencies": {
        "babel-core": "^6.4.0",
        "babel-loader": "^6.2.1",
        "babel-preset-es2015": "^6.3.13",
        "babel-preset-react": "^6.3.13",
        "copy-webpack-plugin": "^1.1.1",
        "css-loader": "^0.23.1",
        "exports-loader": "^0.6.2",
        "expose-loader": "^0.7.1",
        "file-loader": "^0.8.5",
        "image-webpack-loader": "^1.6.2",
        "react-hot-loader": "^1.3.0",
        "style-loader": "^0.13.0",
        "webpack": "^1.12.11",
        "webpack-dev-server": "^1.14.1"
    }
}
{% endhighlight %}

webpack.config.js
{% highlight javascript %}
var webpack = require('webpack');
var path = require('path');
var CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = function(options) {
    var plugins = [
        new webpack.NoErrorsPlugin()
    ];
    var outputFileSuffix = '.entry.js';
    var chunkFileSuffix = ".bundle.js";
    var resolve = {
        extensions: ['', '.js', '.jsx']
    };

    var outputFolder = path.join(__dirname, "../WebContent");

    var debug = true;
    var devtool = "source-map";

    if (options.production) {
        /* minification */
        plugins.push(new webpack.optimize.UglifyJsPlugin({
            minimize: true
        }));
        plugins.push(new webpack.DefinePlugin({
            "process.env": {
                // This has effect on the react lib size
                "NODE_ENV": JSON.stringify("production")
            }
        }));

        debug = false;
        devtool = null;
    } else {
        plugins.push(new webpack.HotModuleReplacementPlugin());
    }

    return {
        entry: {
            index: './src/js/index'
        },
        debug: debug,
        devtool: devtool,
        output: {
            path: outputFolder,
            filename: "javascript/[name]" + outputFileSuffix,
            chunkFilename: "javascript/[id]" + chunkFileSuffix,
            publicPath: './'
        },
        module: {
            loaders: [{
                test: require.resolve("jquery"),
                loader: "expose?$!expose?jQuery"
            }, {
                test: /\.(ttf|eot|svg|woff(2)?)(\?[a-z0-9]+)?$/,
                loader: 'file?name=fonts/[name].[ext]',
                exclude: /(node_modules|node)/
            }, {
                test: /\.css$/,
                loaders: ["style-loader", "css-loader"],
                exclude: /(node_modules|node)/
            }, {
                test: /\.jsx?$/,
                loaders: ['react-hot', 'babel?presets[]=react,presets[]=es2015,compact=false'],
                exclude: /(node_modules|node)/
            }, {
                test: /\.(jpe?g|png|gif|svg)$/i,
                loaders: [
                    'file?name=img/[name].[ext]',
                    'image-webpack?bypassOnDebug&optimizationLevel=7&interlaced=false'
                ],
                exclude: /(node_modules|node)/
            }]
        },
        resolve: resolve,
        plugins: plugins
    };
};

{% endhighlight %}
