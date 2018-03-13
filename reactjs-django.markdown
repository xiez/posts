# 搭建 Django 和 ReactJS 开发环境 #


### 准备环境 ###

#### 前端： ####

- nodejs & npm

  参照[这里](https://docs.npmjs.com/getting-started/installing-node)下载安装。完成后，运行命令查看相应版本。

    ```
    $ node -v
    v9.6.1

    $ npm -v
    5.7.1
    ```

- [webpack](https://webpack.js.org/concepts/)

  前端项目模块（css，js，img）管理工具，把项目所有依赖的库打包到一个或多个bundle文件（例如，bundle.js）。

- [babel](https://babeljs.io/)

  Javascript 编译器，能把新的 ES6 语法转换成 ES5 语法，兼容现有的浏览器。

- [webpack-bundle-tracker](https://github.com/ezhome/webpack-bundle-tracker)

  把 webpack 编译过程记录到文件（webpack-stats.json），供 `django-webpack-loader` 使用。

#### 后端： ####

- [Django V1.11](https://www.djangoproject.com/start/overview/)

        pip install Django==1.11

- [django-webpack-loader](https://github.com/ezhome/django-webpack-loader)


        pip install django-webpack-loader


### 开始流程 ###

- 创建 Django 项目

        $ django-admin startproject django_react_proj
        $ tree django_react_proj
        django_react_proj
        ├── django_react_proj
        │   ├── __init__.py
        │   ├── settings.py
        │   ├── urls.py
        │   └── wsgi.py
        └── manage.py
         
        1 directory, 5 files

  - `$ python manage.py runserver`

     ![django project img](./images/runserver.jpg "python manage.py runserver")

- 增加 index 页面

    - `django_react_proj/urls.py`

     ```
     from django.conf import settings
     from django.conf.urls import url
     from django.contrib import admin
     from django.views.generic import TemplateView
      
     urlpatterns = [
         url(r'^admin/', admin.site.urls),
         url(r'^$', TemplateView.as_view(template_name="index.html")),
     ]
      
     if settings.DEBUG:
         from django.contrib.staticfiles import views
         urlpatterns += [
             url(r'^static/(?P<path>.*)$', views.serve),
         ]
     ```

    - `django_react_proj/settings.py`

        ```
         
        TEMPLATES = [
            {
                'BACKEND': 'django.template.backends.django.DjangoTemplates',
                'DIRS': [os.path.join(BASE_DIR, 'django_react_proj/templates'), ],
                'APP_DIRS': True,
                'OPTIONS': {
                ...
                },
            },
        ]
        ```

    - `django_react_proj/templates/index.html`

       ```
        
       <!DOCTYPE html>
       <html>
           <head>
               <meta charset="UTF-8">
               <title>Example</title>
           </head>
        
           <body>
               <div><p>Hello Django</p></div>
           </body>
       </html>
      ```

    最后效果

    ![hello django](./images/hellodjango.jpg "hello django")

- 创建 React 项目

  在项目根目录下，新建 `frontend` 目录，使用 `npm init` 创建前端项目

        mkdir frontend && cd frontend && npm init

  安装 React 开发相关的依赖

        npm install --save-dev babel-cli babel-loader babel-preset-env babel-preset-react babel-preset-stage-0 react react-dom react-hot-loader webpack webpack-cli webpack-bundle-tracker webpack-dev-server

  创建 webpack config

        mkdir -p assets/js
        touch webpack.config.js
        touch assets/js/index.js

  修改 webpack.config.js

        var path = require("path")
        var webpack = require('webpack')
        var BundleTracker = require('webpack-bundle-tracker')
         
        module.exports = {
          entry: [
            './assets/js/index'
          ],
         
          resolve: {
            extensions: ['.js', '.jsx']
          },
         
          output: {
            path: path.resolve('./assets/bundles/'),
            filename: "bundle.js",
            sourceMapFilename: 'bundle.map',
          },
         
          devtool: '#source-map',
         
          module: {
            rules: [
              {
                test: /\.jsx?$/,
                exclude: /(node_modules)/,
                loader: 'babel-loader',
                query: {
                  presets: ['env', 'stage-0', 'react']
                }
              }
            ]
          },
          
          context: __dirname,
         
          plugins: [
            new BundleTracker({filename: './webpack-stats.json'})
          ],
         
          mode: 'development'
        }

  现在的目录结构：

        $ tree -L 2
        .
        ├── db.sqlite3
        ├── django_react_proj
        │   ├── __init__.py
        │   ├── __init__.pyc
        │   ├── settings.py
        │   ├── settings.pyc
        │   ├── urls.py
        │   ├── urls.pyc
        │   ├── wsgi.py
        │   └── wsgi.pyc
        ├── frontend
        │   ├── assets
        │   ├── node_modules
        │   ├── package-lock.json
        │   ├── package.json
        │   └── webpack.config.js
        └── manage.py



  使用 webpack 编译前端项目 `$ ./node_modules/.bin/webpack-cli --config webpack.config.js`。完成后生成的文件位于 `assets/bundles/bundle.js`

- 修改 Django settings，增加 `webpack_loader` 配置

  - `settings.py`

      ```
      STATICFILES_DIRS = (
          os.path.join(BASE_DIR, 'frontend/assets'),  # We do this so that django's collectstatic copies or our bundles to the STATIC_ROOT or syncs them to whatever storage we use.
      )
       
      WEBPACK_LOADER = {
          'DEFAULT': {
              'BUNDLE_DIR_NAME': 'bundles/',
              'STATS_FILE': os.path.join(BASE_DIR, 'frontend/webpack-stats.json'),
          }
      }
       
      # Application definition
       
      INSTALLED_APPS = [
          ...
          'webpack_loader',
      ]
      ```

  - `templates/index.html`

     ```
     {% load render_bundle from webpack_loader %}
     <!DOCTYPE html>
     <html>
         <head>
             <meta charset="UTF-8">
             <title>Example</title>
         </head>
      
         <body>
             <div><p>Hello Django</p></div>
             <div id="react-app"></div>
             {% render_bundle 'main' %}
         </body>
     </html>
     ```

    刷新页面后，浏览器会去加载`frontend/asserts/bundles/bundle.js`

    ![bundlejs](./images/bundlejs.jpg "bundlejs")


- 修改 React App，增加 App 组件

  - `frontend/assets/js/index.js`

     ```
     import React from 'react';
     window.React = React;
     import ReactDOM from 'react-dom';
     import App from './App';
      
     ReactDOM.render(<App/>,
                     document.getElementById('react-app'))
     ```

  - `frontend/assets/js/App.jsx`

     ```
     const App = () =>
               <p>Hello ReactJS</p>;
      
     export default App;          
     ```

  重新编译

        $ ./node_modules/.bin/webpack-cli --config webpack.config.js

  成功后，刷新页面

  ![reactjs](./images/reactjs.jpg "reactjs")

  `webpack` 增加`watch`选项，自动编译。后续任何 React 模块的改动，webpack 都会自动编译，刷新浏览器页面即可看到改动效果，加快开发效率。

    `./node_modules/.bin/webpack-cli --config webpack.config.js --watch`



### webpack 模块热替换 HMR ###

使用 [webpack HMR](https://webpack.js.org/concepts/hot-module-replacement/) 功能，可以在不刷新浏览器页面的情况下，将 React 模块的改动自动应用到浏览器页面上，进一步加快开发效率。

- 安装 `babel-preset-react-hot`

        npm install babel-preset-react-hot --save-dev


- 修改 `webpack.config.js`

```
var path = require("path")
var webpack = require('webpack')
var BundleTracker = require('webpack-bundle-tracker')

module.exports = {
  entry: [
    'webpack-dev-server/client?http://localhost:8080',
    'webpack/hot/only-dev-server',
    './assets/js/index'
  ],

  resolve: {
    extensions: ['.js', '.jsx']
  },

  output: {
    path: path.resolve('./assets/bundles/'),
    filename: "bundle.js",
    sourceMapFilename: 'bundle.map',
    publicPath: 'http://localhost:8080/assets/bundles/', // Tell django to use this URL to load packages and not use STATIC_URL + bundle_name

  },

  devtool: '#source-map',

  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /(node_modules)/,
        loader: 'babel-loader',
        query: {
          presets: ['env', 'stage-0', 'react', 'react-hot']
        }
      }
    ]
  },

  context: __dirname,

  plugins: [
    new BundleTracker({filename: './webpack-stats.json'})
  ],

  devServer: {
    headers: {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, PATCH, OPTIONS",
      "Access-Control-Allow-Headers": "X-Requested-With, content-type, Authorization"
    }
  },

  mode: 'development'
}
```

- 启动 `webpack-dev-server`

        $ ./node_modules/.bin/webpack-dev-server --inline --progress --hot

  最终效果

  ![hot reload](./images/hotreload.gif "hot reload")

### 示例代码 ###

<https://github.com/xiez/django_react_proj/>


### 参考链接 ###

<http://owaislone.org/blog/webpack-plus-reactjs-and-django/>

<http://matthewlehner.net/react-hot-module-replacement-with-webpack/>
