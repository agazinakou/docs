---
id: mail
title: Mail
---

- [Introduction](#introduction)
- [Configuration](#configuration)
- [Utilisation](#utilisation)
  - [La méthode send](#la-methode-send)
    - [Prototype send](#prototype-send)
    - [Manipulation de l'instance Message](#manipulation-de-l-instance-message)
  - [La méthode raw](#la-methode-raw)
    - [Prototype raw](#prototype-raw)

## Introduction

Envoyer des emails dans une application est monnaie courante. Bow intègre un système d'envoi d'email facile à utiliser. Vous pouvez utiliser ce système à travers plusieurs pilotes communement appelés drivers:

- **SMTP** - Envoie direct via un serveur SMTP
- **MAIL** - Le système utilisera le fonction `email` natif de PHP

## Configuration

Vous trouverez la configuration dans le fichier `config/mail.php` qui est rélativement simple, les options sont toutes commentés.

## Utilisation

Pour utiliser le service, vous devez appeler la classe [`Bow\Mail\Mail::class`](https://bowphp.github.io/api/master/Bow/Mail). Il y a deux méthode statique pour l'envoie de mail `send` et `raw`.

### La méthode send

**send** permet d'envoyer des emails en se basant sur une [vue](http://bowphp.github.io/docs/views) dans l'application.

#### Prototype send

```php
send($view, array $data, callable $callable)
```

| paramêtre | Type |
|----------|------|
| view | `String` L'email du destinataire |
| data | `Array` Les informations à passer à la vue |
| callable | `Closure` Le constructeur du message |

`$callable` vous permet de construire le message et prend en paramètre une instance de [`Bow\Mail\Message`](https://bowphp.github.io/api/master/Bow/Mail/Message.html) qui permet d'ajouter le destinataire, l'objet, les pièces jointes, l'expéditeur.

Exemple d'utilisation:

Considérons la vue `notification-email.tintin.php` qui contient les informations suivantes:

```twig
Bonjour {{ $name }},

Nous vous informons que vous compte viens d'être créditer de 100.000 F.

Cordialement,
```

```php
use Bow\Mail\Mail;
use Bow\Mail\Message;

$data = [
  'name' => 'Franck',
];

Mail::send('notification-email', $data, function (Message $message) {
  $message->to('info@exemple.com');
  $message->subject("Paiement !");
  $messate->from("Bow <info@exemple.com>");
});
```

#### Manipulation de l'instance Message

Bow va donc passer une instance de `Bow\Mail\Message` qui vous permettra d'ajouter des informations supplémentaires à votre email, notamment le destinataire.

`addHeader`, ajout des entêtes personnalisés

```php
addHeader(string $key, string $value)
```

`to`, définit le destinataire

```php
to(string $to, string $name = null)
```

`toList`, Ajout plusieurs destinataires

```php
toList(array $list_desc)
```

`addFile`, Permet d'ajouter une pièce jointe

```php
addFile(string $file)
```

`subject`, Définit l'objet du mail

```php
subject(string $subject)
```

`from`, définit l'expéditeur du mail

```php
from(string $from, string $name = null)
```

`toHtml`, définir le type de contenu en text/html

```php
html($html)
```

`toText`, Définit le corps du message

```php
text(string $text)
```

`addBcc`, ajout l'entête Carbon Copy

```php
addBcc(string $mail, string $name = null)
```

`addCc`, ajout l'entête Carbon Copy

```php
addCc(string $mail, string $name = null)
```

`addReplyTo`, ajout l'entête Reply-To

```php
addReplyTo($mail, $name = null)
```

`addReturnPath`, ajout l'entête Return-Path

```php
addReturnPath($mail, $name = null)
```

### La méthode raw

**raw** par contre, permet d'envoyer des emails avec un message brute

#### Prototype raw

```php
raw($to, $subject, $message, array $headers = [])
```

| paramêtre | Type |
|----------|------|
| to | `String` L'email du destinataire |
| subject | `String` L'objet du mail |
| message | `String` Le message à envoyer |
| headers | `Array` Les entête-https à envoyer, par defaut est vide |

```php
use Bow\Mail\Mail;

$email = "hello@exemple.com";
$subject = "Hello, world";
$message = "C'est juste un email de teste";

Mail::raw($email, $subject, $message);
```

> N'hésitez pas à donner votre avis sur la qualité de la documentation ou proposez des correctifs.
