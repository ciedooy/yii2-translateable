# Translateable Behavior for Yii 2

[![Packagist Version](https://img.shields.io/packagist/v/ciedooy/yii2-translateable.svg?style=flat-square)](https://packagist.org/packages/ciedooy/yii2-translateable)

A modern translateable behavior for the Yii framework.

## Installation

The preferred way to install this extension is through [composer](http://getcomposer.org/download/).

Either run

```bash
$ composer require ciedooy/yii2-translateable
```

or add

```
"ciedooy/yii2-translateable": "~1.0"
```

to the `require` section of your `composer.json` file.

## Migrations

Run the following command

```bash
$ yii migrate/create create_post_table
```

Open the `/path/to/migrations/m_xxxxxx_xxxxxx_create_post_table.php` file,
inside the `up()` method add the following

```php
$this->createTable('{{%post}}', [
    'id' => Schema::TYPE_PK,
]);
```

Run the following command

```bash
$ yii migrate/create create_post_translation_table
```

Open the `/path/to/migrations/m_xxxxxx_xxxxxx_create_post_translation_table.php` file,
inside the `up()` method add the following

```php
$this->createTable('{{%post_translation}}', [
    'post_id' => Schema::TYPE_INTEGER . ' NOT NULL',
    'language' => Schema::TYPE_STRING . '(16) NOT NULL',
    'title' => Schema::TYPE_STRING . ' NOT NULL',
    'body' => Schema::TYPE_TEXT . ' NOT NULL',
]);

$this->addPrimaryKey('', '{{%post_translation}}', ['post_id', 'language']);
```

## Configuring

Configure model as follows

```php
use creocoder\translateable\TranslateableBehavior;

/**
 * ...
 * @property string $title
 * @property string $body
 * ...
 */
class Post extends \yii\db\ActiveRecord
{
    public function behaviors()
    {
        return [
            'translateable' => [
                'class' => TranslateableBehavior::className(),
                'translationAttributes' => ['title', 'body'],
                // translationRelation => 'translations',
                // translationLanguageAttribute => 'language',
            ],
        ];
    }

    public function transactions()
    {
        return [
            self::SCENARIO_DEFAULT => self::OP_ALL,
        ];
    }

    public function getTranslations()
    {
        return $this->hasMany(PostTranslation::className(), ['post_id' => 'id']);
    }
}
```

Model `PostTranslation` can be generated using Gii.

With the subsequent removal  `post_id` from `required` rules
```php
   /**
     * @inheritdoc
     */
    public function rules()
    {
        return [
            [[ /* 'post_id', !!! */ 'language'], 'required'],
            [['post_id'], 'integer'],
            [['language'], 'string', 'max' => 15],
            // .....            
];
    }
```

## Usage

### Setting translations to the entity

To set translations to the entity

```php
$post = new Post();

// title attribute translation for default application language
$post->title = 'Post title';

// body attribute translation for default application language
$post->body = 'Post body';

// title attribute translation for German
$post->translate('de-DE')->title = 'Post titel';

// body attribute translation for German
$post->translate('de-DE')->body = 'Post inhalt';

// title attribute translation for Russian
$post->translate('ru-RU')->title = 'Заголовок поста';

// body attribute translation for Russian
$post->translate('ru-RU')->body = 'Тело поста';

// save post and its translations
$post->save();
```

### Getting translations from the entity

To get translations from the entity

```php
$posts = Post::find()->with('translations')->all();

foreach ($posts as $post) {
    // title attribute translation for default application language
    $title = $post->title;

    // body attribute translation for default application language
    $body = $post->body;

    // title attribute translation for German
    $germanTitle = $post->translate('de-DE')->title;

    // body attribute translation for German
    $germanBody = $post->translate('de-DE')->body;

    // title attribute translation for Russian
    $russianTitle = $post->translate('ru-RU')->title;

    // body attribute translation for Russian
    $russianBody = $post->translate('ru-RU')->body;
}
```

### Checking for translations in the entity

To check translations in the entity

```php
$post = Post::findOne(1);

// checking for default application language translation
$result = $post->hasTranslation();

// checking for German translation
$result = $post->hasTranslation('de-DE');

// checking for Russian translation
$result = $post->hasTranslation('ru-RU');
```

## Advanced usage

### Collecting tabular input

Example of controller actions

```php
class PostController extends Controller
{
    public function actionCreate()
    {
        $model = new Post();

        //...
    }

    public function actionUpdate($id)
    {
        $model = Post::find()->with('translations')->where(['id' => $id])->one();

        if ($model === null) {
            throw new NotFoundHttpException('The requested page does not exist.');
        }

        //...
    }
}
```

Example of view form

```php
<?php
use yii\helpers\Html;
use yii\widgets\ActiveForm;

$form = ActiveForm::begin();

foreach (['en-US', 'de-DE', 'ru-RU'] as $language) {
    echo $form->field($model->translate($language), "[$language]title")->textInput();
    echo $form->field($model->translate($language), "[$language]body")->textarea();
}

//...

ActiveForm::end();
```

### Language specific translation attribute labels

Example of model attribute labels

```php
class PostTranslation extends \yii\db\ActiveRecord
{
    public function attributeLabels()
    {
        switch ($this->language) {
            case 'de-DE':
                return [
                    'title' => 'Titel',
                    'body' => 'Inhalt',
                ];
            case 'ru-RU':
                return [
                    'title' => 'Заголовок',
                    'body' => 'Тело',
                ];
            default:
                return [
                    'title' => 'Title',
                    'body' => 'Body',
                ];
        }
    }
}
```

## Donating

Support this project and [others by creocoder](https://gratipay.com/creocoder/) via [gratipay](https://gratipay.com/creocoder/).

[![Support via Gratipay](https://cdn.rawgit.com/gratipay/gratipay-badge/2.3.0/dist/gratipay.svg)](https://gratipay.com/creocoder/)
