---
id: tintin
title: Tintin Template
---

<h1 align="center">
    <img src="https://github.com/bowphp/arts/raw/master/tintin.jpeg" width="200px" style="border-radius: 50px">
</h1>

- [Introduction](#introduction)
- [Installation](#installation)
- [Configuration](#configuration)
- [Utilisation](#utilisation)
  - [Configuration pour Bow](#configuration-pour-bow)
  - [Ajouter un commentaire](#ajouter-un-commentaire)
  - [#if / #elseif ou #elif / #else](#if--elseif-ou-elif--else)
  - [#unless](#unless)
  - [#loop / #for / #while](#loop--for--while)
    - [L'utilisation de #loop](#lutilisation-de-loop)
    - [Les sucres syntaxiques #jump et #stop](#les-sucres-syntaxiques-jump-et-stop)
    - [L'utilisation de #for](#lutilisation-de-for)
    - [L'utilisation de #while](#lutilisation-de-while)
  - [Inclusion de fichier](#inclusion-de-fichier)
    - [Exemple d'inclusion](#exemple-dinclusion)
- [Héritage avec #extends, #block et #inject](#h%C3%A9ritage-avec-extends-block-et-inject)
  - [Explication](#explication)
- [Directive personnelisée](#directive-personnelis%C3%A9e)
  - [Exemple](#exemple)
  - [Utilisation des directives](#utilisation-des-directives)
  - [Compilation du template](#compilation-du-template)
  - [Sortie après compilation](#sortie-apr%C3%A8s-compilation)
  - [Ajouter vos directive de la configuration](#ajouter-vos-directive-de-la-configuration)
- [Contribution](#contribution)
- [Auteur](#auteur)
- [IDE support](#ide-support)

## Introduction

Tintin est un template PHP qui se veut très simple et extensible. Il peut être utilisable dans n'importe quel projet PHP.

## Installation

Pour installer le package il sera plus mieux utiliser `composer` qui est gestionnaire de package `php`.

```bash
composer require bowphp/tintin
```

## Configuration

Vous pouvez utiliser le package simplement, comme ceci:

```php
require 'vendor/autoload.php';

$tintin = new Tintin\Tintin;

echo $tintin->render('Hello, world {{ strtoupper($name) }}', [
  'name' => 'tintin'
]);
// -> Hello, world TINTIN
```

Mais sauf que cette façon de faire ne permet pas d'exploiter le système d'héritage de façon optimal. Utilisez cette façon de faire, seulement si vous voulez tester le package ou pour les petites applications.

Pour utiliser proprement le package, il faut suivre plutôt la configuration qui suivant:

```php
require 'vendor/autoload.php';

$loader = new Tintin\Loader\Filesystem([
  'path' => '/path/to/the/views/source',
  'extension' => 'tintin.php',
  'cache' => '/path/to/the/cache/directory'
]);

$tintin = new Tintin\Tintin($loader);
```

| Paramètre | Description |
|---------|-------------|
| __php__ | Le chemin vers le dossier des vues de votre applications |
| __extension__ | l'extension des fichiers de template. Par defaut, la valeur est `tintin.php` |
| __cache__ | Le dossier de cache. C'est là que `tintin` va créer le cache. S'il n'est pas défini, `tintin` mettra en cache les fichiers compilés dans le répertoire temporaire de `php`.  |

## Utilisation

```php
// Configuration faite au préalabe
$tintin = new Tintin\Tintin($loader);

$tintin->render('filename', [
  'name' => 'data'
]);
// Ou
$tintin->render('dossier/filename', [
  'name' => 'data'
]);
// Ou
$tintin->render('dossier.filename', [
  'name' => 'data'
]);
```

> Notez que la source des fichiers est toujour le chemin vers `path`.

### Configuration pour Bow

Pour permet à Bow d'utiliser Tintin comme moteur de template par defaut, il va faloir faire quelque petit configuration.

Ajouter cette configuration dans le fichier `app/Kernel/Loader.php`:

```php
public function configurations() {
  return [
    ...
    \Tintin\Bow\TintinConfiguration::class,
    ...
  ];
}
```

Et encore dans le fichier de configuration des vues situés dans `config/view.php`.

```php
return [
  // Définir le moteur à utiliser
  'engine' => 'tintin',

  // Extension de fichier
  'extension' => '.tintin.php'
];
```

Et c'est tout, désormais votre moteur de template par defaut est `tintin` :+1:

> Notez que Tintin est le template par defaut de Bow Framework

### Affichage des données

Considérons le code suivant:

```php
$name = "Tintin";
```

Vous pouvez afficher le contenu de la variable `$name` de la manière suivante:

```c
Hello, {{ $name }}.
```

Bien entendu, vous n'êtes pas limité à afficher le contenu des variables transmises à la vue. Vous pouvez également faire écho aux résultats de toute fonction PHP (Afficher les résultats de toutes les fonctions PHP). En fait, vous pouvez insérer n'importe quel code PHP dans une instruction `{{}}` Tintin:

```html
Hello, {{ strtoupper($name) }}.
```

> Les instructions Tintin `{{}}` sont automatiquement envoyées via la fonction PHP `htmlspecialchars` pour empêcher les attaques [XSS](https://fr.wikipedia.org/wiki/Cross-site_scripting).

#### Affichage des données non échappées

Par défaut, les instructions Tintin `{{}}` sont automatiquement envoyées via la fonction PHP `htmlspecialchars` pour empêcher les attaques [XSS](https://fr.wikipedia.org/wiki/Cross-site_scripting). Si vous ne souhaitez pas que vos données soient protégées, vous pouvez utiliser la syntaxe suivante:

```html
Hello, {{{ $name }}}.
```

### Ajouter un commentaire

Cette clause `{# ... #}` permet d'ajouter un commentaire à votre code `tintin`.

```c
{# Affichage du nom #}
Hello, {{ $name }}
```

### #if / #elseif ou #elif / #else

Ce sont les clauses qui permettent d'établir des branchements conditionnels comme dans la plupart des langages de programmation.

```c
#if ($name == 'Tintin')
  // ..
#elseif ($name == 'Papac')
  // ..
#else
  // ..
#endif
```

> Vous pouvez utiliser `#elif` à la place de `#elseif`.

### #unless

Petite spécificité, le `#unless` quant à lui, il permet de faire une condition inverse du `#if`.
Pour faire simple, voici un exemple:

```c
#unless ($name == 'tintin')
  // ..
#endunless
// Égale
#if (!($name == 'tintin'))
 // ..
#endif
```

### #loop / #for / #while

Souvent vous pouvez être amener à faire des listes ou répétitions sur des éléments. Par exemple, afficher tout les utilisateurs de votre plateforme.

#### L'utilisation de #loop

Cette clause faire exactement l'action de `foreach`.

```c
#loop ($names as $name)
  Bonjour {{ $name }}
#endloop
```

Cette clause peux être aussi coupler avec tout autre clause tel que `#if`.

Un exemple rapide:

```c
#loop ($names as $name)
  #if ($name == 'tintin')
    Bonjour {{ $name }}
    #stop
  #endif
#endloop
```

Vous avez peut-être remarquer le `#stop`, il permet de stoper l'exécution de la boucle. Il y a aussi son conjoint le `#jump`, lui parcontre permet d'arrêter l'exécution à son niveau et de lancer s'exécution du prochain tour de la boucle.

#### Les sucres syntaxiques #jump et #stop

Souvent le dévéloppeur est amené à faire des conditions d'arrêt de la boucle `#loop` comme ceci:

```c
#loop ($names as $name)
  #if ($name == 'tintin')
    #stop
    // Ou
    #jump
  #endif
#endloop
```

Avec les sucres syntaxique, on peut réduire le code comme ceci:

```c
#loop ($names as $name)
  #stop($name == 'tintin')
  // Ou
  #jump($name == 'tintin')
#endloop
```

#### L'utilisation de #for

Cette clause faire exactement l'action de `for`.

```c
#for ($i = 0; $i < 10; $i++)
 // ..
#endfor
```

#### L'utilisation de #while

Cette clause faire exactement l'action de `while`.

```c
#while ($name != 'tintin')
 // ..
#endwhile
```

### Inclusion de fichier

Souvent lorsque vous dévéloppez votre code, vous êtes amener à subdiviser les vues de votre application pour être plus flexible et écrire moin de code.

`#include` permet d'include un autre fichier de template dans un autre.

```c
 #include('filename', data)
```

#### Exemple d'inclusion

Considérons le fichier `hello.tintin.php` suivant:

```c
Hello {{ $name }}
```

Utilisation:

```c
#include('hello', ['name' => 'Tintin'])
// => Hello Tintin
```

## Héritage avec #extends, #block et #inject

Comme tout bon système de templating **Tintin** support le partage de code entre fichier. Ceci permet de rendre votre code flexible et maintenable.

Considérérons le code **Tintin** suivant:

```c
// le fichier `layout.tintin.php`
<!DOCTYPE html>
<html>
<head>
  <title>Hello, world</title>
</head>
<body>
  <h1>Page header</h1>
  <div id="page-content">
    #inject('content')
  </div>
  <p>Page footer</p>
</body>
</html>
```

Et aussi, considérons un autre fichier qui héritera du code du fichier `layout.tintin.php`

```c
// Le fichier se nomme `content.tintin.php`
#extends('layout')

#block('content')
  <p>This is the page content</p>
#endblock
```

### Explication

Le fichier `content.tintin.php` va hérité du code de `layout.tintin.php` et si vous rémarquez bien, dans le fichier `layout.tintin.php` on a la clause `#inject` qui a pour paramètre le nom du `#block` de `content.tintin.php` qui est `"content"`. Ce qui veut dire que le contenu du `#block` `"content"` sera remplacé par `#inject`. 

Ce qui donnera à la fin ceci:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Hello, world</title>
</head>
<body>
  <h1>Page header</h1>
  <div id="page-content">
    <p>This is the page content</p>
  </div>
  <p>Page footer</p>
</body>
</html>
```

## Directive personnelisée

Tintin peut être étendu avec son systême de directive personnalisé, pour se faire, utilisez la méthode `directive`

```php
$tintin->directive('hello', function (array $attributes = [])
{
  return 'Hello, '. $attributes[0];
});

echo $tintin->render('#hello("Tintin")');
// => Hello, Tintin
```

### Exemple

Création de directive pour gérer un formulaires:

```php
$tintin->directive('input', function (array $attributes = [])
{
  $attribute = $attributes[0];

  return '<input type="'.$attribute['type'].'" name="'.$attribute['name'].'" value="'.$attribute['value'].'" />';
});

$tintin->directive('textarea', function (array $attributes = [])
{
  $attribute = $attributes[0];

  return '<textarea name="'.$attribute['name'].'">"'.$attribute['value'].'"</textarea>';
});

$tintin->directive('button', function (array $attributes = [])
{
  $attribute = $attributes[0];

  return '<button type="'.$attribute['type'].'">'.$attribute['label'].'"</button>';
});

$tintin->directive('form', function (array $attributes = [])
{
  $attribute = " ";
  
  if (isset($attributes[0])) {
    foreach ($attributes[0] as $key => $value) {
      $attribute .= $key . '="'.$value.'" ';
    }
  }

  return '<form "'.trim($attribute).'">';
});

$tintin->directive('endform', function (array $attributes = [])
{
  return '</form>';
});
```

### Utilisation des directives

Pour utiliser ces directives, rien de plus simple. Ecrivez le nom de la directive précédé la par `#`. Ensuite si cette directive prend des paramètres, lancer la directive comme vous lancez les fonctions dans votre programme. Les paramètres seront régroupés dans la varibles `$attributes` dans l'ordre d'ajout.

```c
// Fichier form.tintin.php
#form(['method' => 'post', "action" => "/posts"])
  #input(["type" => "text", "value" => null, "name" => "name"])
  #textarea(["value" => null, "name" => "content"])
  #button(['type' => 'submit', 'label' => 'Add'])
#endform
```

### Compilation du template

La compilation se fait comme d'habitude, pour plus d'information sur la [compilation](#utilisation).

```php
echo $tintin->render('form');
```

### Sortie après compilation

```html
<form action="/posts" method="post">
  <input type="text" name="name" value=""/>
  <textarea name="content"></textarea>
  <button type="submit">Add</button>
</form>
```

### Ajouter vos directive de la configuration

Dans le cas ou vous utilisez la configuration Tintin pour Bow Framework.

Vous pouvez créer une classe dans le dossier `app`, par exemple, avec le nom `CustomTintinConfiguration` qui va étendre la configuration par défaut de Tintin qui est `\Tintin\Bow\TintinConfiguration::class` et ensuite modifier la méthode `customizer`.

```php
use Tintin\Tintin;

class CustomTintinConfiguration extends \Tintin\Bow\TintinConfiguration
{
  /**
   * Add action in tintin
   *
   * @param Tintin $tintin
   */
  public function customizer(Tintin $tintin)
  {
    $tintin->directive('super', function (array $attributes = []) {
      return "Super !";
    });
  }
}
```

Maintenant la directive `#super` est disponible et vous pouvez l'utiliser.

```php
  return $tintin->render('#super');
  // => Super !
```

## IDE support

Tintin est supporté actuellement par [sublime text](https://www.sublimetext.com). 

### Comment installer Sublime Package Control ?

Allez sur le [site](https://packagecontrol.io/installation) et suivez les instructions.

### Installer le package Tintin

- Recherchez **Bow Tintin** et installez-le/Téléchargez ou clonez ce référentiel dans [install-dir]/Packages/bow-tintin
- Redémarrez Sublime Text.
- Rouvrez n’importe quel fichier `.tintin` ou `.tintin.php`.
- Enjoy :smiley:
