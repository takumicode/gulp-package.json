# gulp-package.json
gulpのpackage.jsonファイルです。


const { src, dest,lastRun, watch, series, parallel } = require("gulp");
// 直列処理(順番に処理):series(), 並列処理（順番を問わない）:parallel()
const sass = require('gulp-sass');
const plumber = require('gulp-plumber');
const postcss = require('gulp-postcss');
const autoprefixer = require('autoprefixer');
const cssdeclsort = require('css-declaration-sorter');
const cleanCSS = require('gulp-clean-css');//minファイルの作成
const rename = require('gulp-rename');//minファイルの名前変更
const sassGlob = require('gulp-sass-glob'); // @importを纏めて指定
const browserSync = require('browser-sync');
const gcmq = require('gulp-group-css-media-queries'); // media queryを纏める
const imagemin = require('gulp-imagemin');//画像の圧縮
const optipng = require('imagemin-optipng');
const mozjpeg = require('imagemin-mozjpeg');
const mode = require('gulp-mode')({
  modes: ['production', 'development'], // 本番実装時は 'gulp --production'
  default: 'development',
  verbose: false,
})

const compileSass = done => {
  const postcssPlugins = [
    autoprefixer({
    // browserlistはpackage.jsonに記載[推奨]
    cascade: false,
    // grid: 'autoplace' // IE11のgrid対応('-ms-')
    }),
    cssdeclsort({ order: 'alphabetical' /* smacss, concentric-css */ })
  ]

  src("./src/scss/**/*.scss", { sourcemaps: true  /* init */})
  .pipe(plumber())
  .pipe(sassGlob())
  .pipe(sass({outputStyle: 'expanded'}))
  .pipe(postcss(postcssPlugins))
  .pipe(mode.production(gcmq()))
  .pipe(dest("./src/css", { sourcemaps: './sourcemaps' /* write */ }));
  done(); // 終了宣言
}

const mincss = done => {
  src('./src/css/style.css')
  .pipe(cleanCSS())
  .pipe(rename({suffix: '.min'}))
  .pipe(dest('./src/css'));
  done();
}

// ローカルサーバ起動
const buildServer = done => {
  browserSync.init({
    port: 8080,
    notify: false,
    // 静的サイト
    server: {
      baseDir: './',
    },
    // 動的サイト
    // files: ['./**/*.php'],
    // proxy: 'http://localsite.wp/',
  })
  done()
}

//圧縮率の定義
const imageminOption = [
  optipng({ optimizationLevel: 5 }),
  mozjpeg({ quality: 85 }),
  imagemin.gifsicle({
   interlaced: false,
   optimizationLevel: 1,
   colors: 256
}),
imagemin.mozjpeg(),
imagemin.optipng(),
imagemin.svgo()
];

 const imageminAll = done => {
   src('./src/img/base/*.{png,jpg,gif,svg}', { since: lastRun(imageminAll) })
   .pipe(imagemin(imageminOption))
   .pipe(dest('./src/img'));
   done();
 }

// ブラウザ自動リロード
const browserReload = done => {
  browserSync.reload()
  done()
}

// ファイル監視
const watchFiles = () => {
  watch('./**/*.html', browserReload);
  watch('./src/scss/*.scss', browserReload);
  watch('./src/js/**/*.js', browserReload);
  watch('./src/img/base/*.{png,jpg,gif,svg}', browserReload);
}

exports.sass = series(compileSass, mincss, browserReload);
exports.imagemin = series(imageminAll, browserReload);
exports.default = parallel(buildServer, watchFiles);


npm install gulp-sass gulp-plumber gulp-postcss autoprefixer css-declaration-sorter gulp-clean-css gulp-rename gulp-sass-glob browser-sync gulp-imagemin imagemin-optipng imagemin-mozjpeg gulp-mode postcss@8.1.0 --save-dev
