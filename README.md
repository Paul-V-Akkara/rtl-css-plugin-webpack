# rtl-css-plugin-webpack
Webpack plugin to use in addition to [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin) to create a second css bundle, processed to be rtl.

This uses [rtlcss](https://github.com/MohammadYounes/rtlcss) under the hood

## Installation

```shell
$ npm install rtl-css-plugin-webpack
```


## Usage

Add the plugin to your webpack configuration:

```js
const RtlCssPluginWebpack = require('rtl-css-plugin-webpack');

module.exports = {
  entry: path.join(__dirname, 'src/index.js'),
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    chunkFilename: 'chunk/[name].[hash].js',

  },
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [MiniCssExtractPlugin.loader, "css-loader"],
      },
    ],
  },
  plugins: [new MiniCssExtractPlugin(), new RtlCssPluginWebpack()]
};
```

## Options


```
new RtlCssPlugin({
  filename: '[name].rtl.css',
  chunkFilename: '[name].rtl.css',
  });
```
#### `filename`

Default: `[name].rtl.css`

This option determines the name of each output CSS file.

#### `chunkFilename`

Default: `[name].rtl.css`

This option determines the name of non-entry chunk files.


## Example with MiniCSSExtractPlugin to dynamic load rtl css

The insert option of [mini-css-extract-plugin](https://www.npmjs.com/package/mini-css-extract-plugin#insert) can be used to load  non-initial (async) rtl/ltr CSS chunks


> **Note**
>
> It's only applied to dynamically loaded css chunks, if you want to modify link attributes inside html file, please using [html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin)


```js
const RtlCssPluginWebpack = require('rtl-css-plugin-webpack');

module.exports = {
  entry: path.join(__dirname, 'src/index.js'),
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    chunkFilename: 'chunk/[name].[hash].js',

  },
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [MiniCssExtractPlugin.loader, "css-loader"],
      },
    ],
  },
  plugins: [
      // generates ltr css files
      new MiniCssExtractPlugin({
      filename: '[name]-ltr.css',
      chunkFilename: 'chunks/[name].[hash]-ltr.css',
      // this function is going to be part of bundle to load rtl/ltr css chunk based on html dir attribute
      insert: linkTag => {
        if (document.documentElement.dir === 'rtl') {
          linkTag.href = linkTag.href.replace('-ltr', '-rtl');
        }
        document.head.appendChild(linkTag);
      },
    }),
    // generates rtl css files
      new RtlCssPluginWebpack({
            filename: '[name]-rtl.css',
            chunkFilename: 'chunks/[name].[hash]-rtl.css',
      }),
    ]
};
```

## Example for optimization

We can update webpacks optimization config to avoid duplicate css chunks

```js
module.exports = {
  // ... other webpack configs
  optimization: {
    // ... other optimization
    splitChunks: {
      cacheGroups: {
        // avoids duplication of dynamically loaded css chunks
        styles: {
          test: /\.(css|scss|less)$/,
          enforce: true,
        },
      },
    },
  }
}
```
