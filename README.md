# Как выжать максималочку из pagespeed

[Проверяем сайт](https://developers.google.com/speed/pagespeed/insights/)

Следуем рекомендациям 

Все .. ) 
## Шрифты 

Есть два варианта.

1. подключаем с [google fonts](https://fonts.google.com/) в формате `@import url('https://fonts.googleapis.com/css?family=Montserrat:300,400,500,700,900&display=swap&subset=cyrillic')`. 

2. Качаем локально, [генерируем](https://transfonter.org/) каждый из видов в 3-х базовых разширениях, а именно: 
  - **woff2**
  - **woff**
  - **ttf**
  - **eot** // только для IE i Edge(<=18)
```sass
@font-face {
  font-family: Lato
  font-display: swap
  src: url(/static/css/Lato.woff2) format('woff2'),
       url(/static/css/Lato.woff) format('woff'),
       url(/static/css/Lato.eot) format('embedded-opentype'),
       url(/static/css/Lato.ttf) format('truetype')
}
```
Шрифты рекомендую хранить в одной папке со стилями, они быстрее подключатся, без хождения по папочкам.

Обязательно проверяем их наличие в Network.
http://joxi.ru/Dr8XeqLHzBM5Or

Не забываем про `font-display`.

Как видишь, первый вариант быстрее и легче в использовании. При этом немного разгрузим нам css , но добавим время на сторонние ресурсы. Бесследно не пройдет)

## Минифицированный код

Запускаем таску `npm run build`. Исправляем все, до единой , ошибки в `js`. Не забываем подключить `manifest.js`. Для `django` - в базовом шаблоне. Для `wp` - в `functions.php`. 

## Работа с изображениями

Максимально отказываемся от `background-image`. Используя новые свойства `object-fit` i `object-position` мы добиваемся тех же любимых прелестей фона, только на теге `img`. При условии не поддерживания IE i Edge(<=18).

* Если поддерживаем IE i Edge(<=18) - статику, кроме автоматической таски `imagemin` с `build`-а  , прогони, не поленись, через [этот сервис](https://www.iloveimg.com/compress-image). Потому что если еще wp с этим что-то сделает, то джанга точно нет.

### Wordpress
Для `wp` ставим [WebP Express](https://wordpress.org/plugins/webp-express/)

Проверяем [настройки](http://joxi.ru/nAy03yjHjJa1N2)

Запускаем **Bulk Convert**. Ждем довольно долго пока он нагенерирует изображений в формате `webp`.

Ну и [tinyPng](https://wordpress.org/plugins/tiny-compress-images/)

Первым шагом [оптимизируем то, что есть](http://joxi.ru/n2YQ7w4sZN7q02)

Дальше при загрузках - должны оба плагина отработать.

### Django

Контактируем с бекендером, он делает магию и отдает тебе `webP`. Если не захочет делать, упорно настаиваем. Он обязан это сделать) Обращаемся к Олеже, вдруг у кого проблемы с генерацией, он поможет.

```pug
+b.tt-picture--size_auto.PICTURE()
  source(
    type="image/webp"
    srcset!=exp("variant.sized_image.catalog_preview_webp")
  )
  +e.body.IMG(
    src!=exp('variant.sized_image.catalog_preview'),
    alt!=exp('variant.image_alt'),
  )
```

Место для картинки в формате `webP` размечаем подобным образом, для fallbackа в браузерах, которые не поддерживают данный формат.

### Использование srcset

Важным пунктом в pageSpeed на моб.версии является использование изображений правильного размера. Имеется ввиду, что вставить на моб версию картинку с разрешением `1200*500` не целесообразно. Тут нам на помощь придет `scrset`.

```pug
img(sizes='(min-width: 1440px) 1440w, (min-width: 1200px) 1200w, min-width: 320px) 320w',
  srcset=exp("item.img_array.sizes.gallery_small")+ " 320w, "+exp("item.img_array.sizes.gallery_middle")+ " 1200w, "+exp("item.img")+ " 1440w", ,
  src!=exp("item.img")
)
```
Суть в чем.

Атрибут `sizes` позволяет явно определить размер изображения, который должен быть использован, в зависимости от условий media.

Атрибут `srcset` позволяет использовать комплект исходных файлов изображения, которые могут быть использованы при рендеринге страницы в браузере. Для этого нужно задать список изображений, разделенных запятыми, и юзер-агент определяет, какое из них отобразить, в зависимости от типа и особенностей устройства.

Задаем в сайзах, столько брейпоинтов - сколько хотите. Как минимум , для эффективности , их должно быть 2. Для моб и для десктопа.

А в сете - уже указываем какая картинка - принадлежит какому сайзу. На резайз  эта штука не отработает и src она явно не изменяет, увидеть, применились ли изменения, можно лишь наведя на [src](http://joxi.ru/Grqb6RNCkLzo9m). Мы тут увидим `currentSrc`.


Размеры опять согласовываем с бекендером и он нарезает такие, какие тебе надо. 

## Preload

При необходимости указываем предзагрузку статических файлов.

```pug
link( rel='preload', as='font',  type="font/woff" crossorigin="anonymous" href!=static("'css/materialdesignicons-webfont.woff2'"))
link( rel='preload', as='font', href!=static("'fonts/svgfont.woff'")  type="font/woff2" crossorigin="anonymous")
link( href!=url('"vuejs_translate:js-i18n"'), rel="preload", as='script')
link( href!=static("'js/manifest.js'"), rel="preload", as='script')
```

## Доп фичи

### Скажи НЕТ прелоадеру.

Отказываемся. Пейдж Спид жутко не любит. Он накручивает почти все пункты загрузки старницы.

### Никаких lazyload плагинов.
Максималочка - атрибут `lazyload`. Лишние скрипты никому не нужны, у нас их и так ооочень много, а оптимизированные изображения в формате `WEBP` никогда не потребуют `lazyload`.

