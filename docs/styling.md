title: Styling
template: extrahead.html

Modify javascript / scss styling


### npm

Install npm for all the dependencies you need, for that you need to be in the root folder of that project.

```
npm install
```

In that `webpack.mix.js` <a href="https://github.com/Devsome/silkroad-laravel/blob/master/webpack.mix.js">link</a>
you can see that original source and the compressed folder path.

If you want to debug and run that stuff in development mode run this
```
npm run dev
```

else you can run
```
npm run production
```

There is also a third method when you are currently working on a design and want to refresh it everytime, then you can run:
```
npm run watch
```
so there is a watcher running and everytime you change a javascript or scss file it will be automatically generated.


### Logo

at `public/image/logo/logo.png` you can change the project logo. It used in the E-Mail Templates.


If you want some help, check the <a href="https://laravel.com/docs/6.x/mix">Laravel Compiling Assets (Mix)</a><br>
Hopefully you are good to go with that.
