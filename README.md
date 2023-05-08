# Up and running with Gulp and NPM

Gulp and NPM setup for front-end web development.

## NPM

NPM is the package manager for JavaScript.

### How to Install

NPM is distributed with Node.js, which means that when we download Node.js, we automatically get npm installed on our computer.

- [Node.js](https://nodejs.org/en/)

## GULP

GULP is a toolkit for automating painful or time-consuming tasks in your development.

### How to Install

Install the gulp command line utility: `npm install --global gulp-cli`

- [Quick Start guide for Gulp installation](https://gulpjs.com/docs/en/getting-started/quick-start)

## NPM packages (sample)

- [yargs](https://www.npmjs.com/package/yargs) - yargs helps you build interactive command line tools.
- [gulp-sass](https://www.npmjs.com/package/gulp-sass) - sass plugin for gulp.
- [gulp-clean-css](https://www.npmjs.com/package/gulp-clean-css) - gulp plugin to minify css, using clean-css.
- [gulp-if](https://www.npmjs.com/package/gulp-if) - a ternary gulp plugin, conditionally control the flow of vinyl objects.
- [gulp-sourcemaps](https://www.npmjs.com/package/gulp-sourcemaps) - write inline source maps.
- [gulp-imagemin](https://www.npmjs.com/package/gulp-imagemin) - minify PNG, JPEG, GIF and SVG images with imagemin.
- [del](https://www.npmjs.com/package/del) - delete files and folders using globs
- [webpack-stream](https://www.npmjs.com/package/webpack-stream) - run webpack as a stream to conveniently integrate with gulp.
- [babel-loader](https://www.npmjs.com/package/babel-loader) - this package allows transpiling JavaScript files using Babel and webpack.
- [gulp-uglify](https://www.npmjs.com/package/gulp-uglify) - minify JavaScript with UglifyJS3.
- [vinyl-named](https://www.npmjs.com/package/vinyl-named) - give vinyl files arbitrary chunk names.
- [gulp-zip](https://www.npmjs.com/package/gulp-zip) - ZIP compress files.
- [gulp-replace](https://www.npmjs.com/package/gulp-replace) - a string replace plugin for gulp 3.

## Use latest JavaScript version in our gulpfile

Node already supports a lot of ES2015+ features, but to avoid compatibility problems we need to **install [Babel](https://babeljs.io/docs/en/babel-register)** and rename our **_gulpfile.js_** as **_gulpfile.babel.js_**.

`npm install --save-dev @babel/register @babel/core @babel/preset-env`

Then create a **_.babelrc_** file with the preset configuration.

## Sample .babelrc

```js
{
    "presets": [ "@babel/preset-env" ]
}
```

## Sample gulp.babel.js

This is our **_gulp.babel.js_** written in ES2015+ syntax with npm packages we use.

```js
import gulp from 'gulp';
import yargs from 'yargs';
import cleancss from 'gulp-clean-css';
import gulpif from 'gulp-if';
import sourcemaps from 'gulp-sourcemaps';
import imagemin from 'gulp-imagemin';
import del from 'del';
import webpack from 'webpack-stream';
import uglify from 'gulp-uglify';
import named from 'vinyl-named';
import zip from 'gulp-zip';
import replace from 'gulp-replace';
import merge from 'merge-stream';

import info from './package.json';

const PRODUCTION = yargs.argv.prod;

const paths = {
   styles: {
	src: ['src/assets/css/bundle.css', 'src/assets/css/responsive.css'],
	dest: 'dist/assets/css'
   },
   images: {
	src: 'src/assets/images/**/*.{jpg,jpeg,png,svg,gif}',
	dest: 'dist/assets/images'
   },
   scripts: {
	src: 'src/assets/js/bundle.js',
	dest: 'dist/assets/js'
   },
   other: {
	src: ['src/assets/**/*', '!src/assets/{images,js,css}', '!src/assets/{images,js,css}/**/*'], 
	dest: 'dist/assets'
   },
   packaged: {
	src: ['**/*', '!.vscode', '!node_modules{,/**}', '!packaged{,/**}', '!src{,/**}', '!.babelrc', '!.gitignore', '!gulpfile.babel.js', '!package.json', '!package-lock.json'],
	dest: 'packaged'
   }
}

// Default task
exports.default = (done) => {
   console.log('gulp works!')
   done();
}

export const styles = () => {
   return gulp.src(paths.styles.src)
	.pipe(gulpif(!PRODUCTION, sourcemaps.init()))
	.pipe(gulpif(PRODUCTION, cleancss({ 'compatability': 'ie8' })))
	.pipe(gulpif(!PRODUCTION, sourcemaps.write()))
	.pipe(gulp.dest(paths.styles.dest));
}

export const images = () => {
   return gulp.src(paths.images.src)
	.pipe(gulpif(PRODUCTION, imagemin()))
	.pipe(gulp.dest(paths.images.dest));
}

export const scripts = () => {
   return gulp.src(paths.scripts.src)
	.pipe(named())
        .pipe(webpack({
            module: {
                rules: [{
		    test: /\.js$/,
		    use: { 
		    	loader: 'babel-loader',
			options: {
				presets: ['@babel/preset-env']
			}
		    }
		}]
            },
	    output: {
	        filename: '[name].js'
	    },
	    devtool: !PRODUCTION ? 'inline-source-map' : false,
            mode: PRODUCTION ? 'production' : 'development' //add this
   }))
   .pipe(gulpif(PRODUCTION, uglify())) //you can skip this now since mode will already minify
   .pipe(gulp.dest(paths.scripts.dest));
}

// Copy third party libraries from /node_modules into /dist/vendor
export const vendors = () => {
   return merge([
	'jquery/dist',
	'bootstrap/dist',
	'font-awesome/{css,fonts}'
   ].map(function (vendor) {
   return gulp.src('node_modules/' + vendor + '/**/*')
	.pipe(gulp.dest('dist/vendors/' + vendor.replace(/\/.*/, '')));
   }));
}

export const copy = () => {
   return gulp.src(paths.other.src)
	.pipe(gulp.dest(paths.other.dest));
}

export const clean = (done) => {
   del(['dist','packaged']).then(paths => {
	console.log('Deleted files and folders:\n', paths.join('\n'));
   });
   done();
}

export const watch = () => {
   gulp.watch('src/assets/css/**/*.css', styles);
   gulp.watch('src/assets/js/**/*.js', scripts);
   gulp.watch(paths.images.src, images);
   gulp.watch(paths.other.src, copy);
}

export const compress = () => {
   return gulp.src(paths.packaged.src)
	.pipe(replace('_projectname', info.name))
	.pipe(zip(`${info.name}.zip`))
	.pipe(gulp.dest(paths.packaged.dest));
}

export const dev = gulp.series(gulp.parallel(styles, images, scripts, vendors, copy), watch);
export const prod = gulp.series(clean, gulp.parallel(styles, images, scripts, vendors, copy));
export const bundle = gulp.series(prod, compress);
```

## GULP commands

These are the commands we define in our **_gulp.babel.js_** file.

`gulp dev`, `gulp prod`, `gulp bundle --prod`, `gulp clean`

---

## License

This project is released under the MIT License.
