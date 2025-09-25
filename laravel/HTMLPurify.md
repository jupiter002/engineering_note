게시글 작성, 수정시 악성 스크립트를 이용한 공격 방지를 할 때 사용

설치방법 :
docker : docker-compose exec <서비스 이름> composer require stevebauman/purify
non-docker : composer require stevebauman/purify

설정 :
config/purify.php
```php
<?php
return [
    /*
     * The default configuration to use.
     */

    'default' => 'default',

    /*
     * The different configurations for HTMLPurifier.
     *
     * You can create multiple configurations, each with its own settings.
     *
     * For more information on the settings, see the HTMLPurifier documentation:
     * http://htmlpurifier.org/live/configdoc/runtime.html
     */

    'configs' => [

  

        'default' => [

            'HTML.Doctype'             => 'HTML 4.01 Transitional',

            'HTML.Allowed'             => 'div,b,strong,i,em,a[href|title],ul,ol,li,p[style],br,span[style],img[width|height|alt|src]',

            'CSS.AllowedProperties'    => 'font,font-size,font-weight,font-style,font-family,text-decoration,padding-left,color,background-color,text-align',

            'AutoFormat.AutoParagraph' => true,

            'AutoFormat.RemoveEmpty'   => true,

        ],

  

        'test' => [

            'HTML.Doctype'             => 'HTML 4.01 Transitional',

            'HTML.Allowed'             => 'div,b,strong,i,em,a[href|title],ul,ol,li,p[style],br,span[style],img[width|height|alt|src]',

            'CSS.AllowedProperties'    => 'font,font-size,font-weight,font-style,font-family,text-decoration,padding-left,color,background-color,text-align',

            'AutoFormat.AutoParagraph' => true,

            'AutoFormat.RemoveEmpty'   => true,
        ],
    ],
];
```
HTML.Allow 항목에 선언한 태그만 허용함.


사용방법
Purify::clean($content)