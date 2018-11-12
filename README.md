## Create react app 2 設定 Semantic UI LESS

#### 建立 react app

<pre>yarn create react-app 專案名稱</pre>

#### 解除 create react app (CRA) 設定

雖然 `CRA 2.0` 開始已內建 `sass` 但卻沒有 `less`，而要設定 `less` 需要改動到 `webpack` 設定，但 `CRA` 為了更容易使用會將所有複雜的設定隱藏， `eject` 指令可將其設定展開，但會無法回到先前的狀態， 但為了使用 `less` 目前必須這麼做。

<pre>yarn eject</pre>

#### 安裝 less
<pre>yarn add less less-loader</pre>

#### 設定 less
##### 1. 修改兩個webpack設定檔
<pre>
config/
    webpack.config.dev.js
    webpack.config.prod.js
</pre>
##### 2. 加入變數
```js
// style files regexes
const cssRegex = /\.css$/;
const cssModuleRegex = /\.module\.css$/;
const sassRegex = /\.(scss|sass)$/;
const sassModuleRegex = /\.module\.(scss|sass)$/;
+ const lessRegex = /\.less$/;
+ const lessModuleRegex = /\.module\.less$/;
```
##### 3. 在module.rules.oneOf內加入規則
```js
// Adds support for CSS Modules, but using SASS
// using the extension .module.scss or .module.sass
{
    test: sassModuleRegex,
    use: getStyleLoaders(
        {
            importLoaders: 2,
            modules: true,
            getLocalIdent: getCSSModuleLocalIdent,
        },
        'sass-loader'
    ),
},
+ //LESS
+ {
+     test: lessRegex,
+     exclude: lessModuleRegex,
+     use: getStyleLoaders({ importLoaders: 2 }, 'less-loader'),
+ },
+ {
+     test: lessModuleRegex,
+     use: getStyleLoaders(
+         {
+             importLoaders: 2,
+             modules: true,
+             getLocalIdent: getCSSModuleLocalIdent,
+         },
+         'less-loader'
+     ),
+ },
```

#### 安裝 semantic

<pre>yarn add semantic-ui-react semantic-ui-react</pre>

#### 設定 semantic less
`semantic less` 會安裝在node_modules內，這會造成客製不易，不管要修改變數或是覆寫內容，所以要將設定值複製到專案資料夾內，但此動作需要再配合一些設定才會連結起來。

##### 1. 建立資料夾 semantic-theme
<pre>
src/
    semantic-theme/
</pre>

##### 2. 複製以下檔案與資料夾至semantic-theme
<pre>
node_modules/
    semantic-ui-less/
        _site/ <--複製整個資料夾並重新命名為site
        semantic.less <--複製此檔案
        theme.config.example <--複製此檔案並重新命名為theme.config
</pre> 

##### 3. 變更 semantic.less 路徑

取代 `definitions` 為 `~semantic-ui-less/definitions` 將路徑指到node_modules底下

##### 4. 變更 theme.config 路徑

##### 4-1. `@siteFolder` 路徑變更為 
<pre>'../../src/semantic-theme/site'</pre>

##### 4-2. `Import Theme` 路徑變更為 
<pre>'~semantic-ui-less/theme.less'</pre>

##### 4-3. 在 `Import Theme Section`下新增一個Section 

由於 semantic ui 使用了 fontawesome 故要將 font 指到正確路徑
<pre>
/*******************************
         Font Path
*******************************/

/* Path to the fonts */
@fontPath : "../../../themes/@{theme}/assets/fonts";
</pre>

#### 設定 webpack

##### 1. `theme.config` 路徑設定

如果現在直接啟動專案會發現他仍找不到 `theme.config` 檔案，每個元件都會`import '../../theme.config'`，但若每個元件都去改這正確位置太麻煩，所以可以靠webpack以別名的方式指定路徑，找到 `resolve.alias` 並新增別名。
```js
alias: {
    // Support React Native Web
    // https://www.smashingmagazine.com/2016/08/a-glimpse-into-the-future-with-react-native-for-web/
    'react-native': 'react-native-web',
+    '../../theme.config$': path.join(__dirname, '../src/semantic-theme/theme.config'),
},
```

##### 2. 排除檔案設定

如果現在直接啟動專案會發現一樣找不到 `theme.config` 檔案，這是因為有內建將檔案以hash方式重新命名引用，為了有變更檔案內容時檔名不被瀏覽器已存在的快取取代而沒有更新，所以 `theme.config` 會變成 `theme.[hash].config`，這樣當然元件會引用不到檔案，所以要將某些附檔名排除，找到 `modules.rules.oneOf` 底下 `file-loader 設定做修改。

```js
{
    // Exclude `js` files to keep "css" loader working as it injects
    // its runtime that would otherwise be processed through "file" loader.
    // Also exclude `html` and `json` extensions so they get processed
    // by webpacks internal loaders.
U   exclude: [/\.(js|mjs|jsx|ts|tsx)$/, /\.html$/, /\.json$/, /\.(config|overrides|variables)$/],
    loader: require.resolve('file-loader'),
    options: {
        name: 'static/media/[name].[hash:8].[ext]',
    },
},
```

#### 完成 semantic ui 設定

接下來可以在 `index.js` 引用 `src/semantic-theme/semantic.less` 做測試，
成功匯入後，可以修改 `src/semantic-theme/site/globals/site.variables` 去做變數覆蓋的測試，變數的設定可以參考 `node_modules/semantic-ui-less/themes/default/globals/site.variables`

範例 `site.variables`
```css
/*******************************
     User Global Variables
*******************************/

@primaryColor        : #DB2828; /*primary button 會變紅色*/
```
