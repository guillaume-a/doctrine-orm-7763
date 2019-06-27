# Create DB and play this sql

```
CREATE TABLE `unicorn` (
  `id` int(10) UNSIGNED NOT NULL,
  `name` varchar(40) NOT NULL COMMENT 'enter a "comment" here'
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
``` 

# Install project

```
composer install
cp .env .env.local
vi .env.local #edit DATABASE_URL
```

# Try to import schema

```
bin/console doctrine:mapping:import "App\Entity" annotation --path=src/Entity
```

# Try to generate getter/setters

```
bin/console make:entity --regenerate App
```

And get the following exception

```
In AnnotationException.php line 42:
                                                                                 
  [Syntax Error] Expected Doctrine\Common\Annotations\DocLexer::T_CLOSE_CURLY_B  
  RACES, got 'comment' at position 122 in property App\Entity\Unicorn::$name
```

# Take a look at generated code

Quotes in comment are not escaped.

```
// src/Entity/Unicorn.php
/**
 * @var string
 *
 * @ORM\Column(name="name", type="string", length=40, nullable=false, options={"comment"="enter a "comment" here"})
 */
private $name;
```

# Diving in the code

Comment generation is made in vendor/doctrine/orm/lib/Doctrine/ORM/Tools/EntityGenerator.php in the generateFieldMappingPropertyDocBlock function.
On line 1681 there's this code
```
if (isset($fieldMapping['options']['comment']) && $fieldMapping['options']['comment']) {
    $options[] = '"comment"="' . $fieldMapping['options']['comment'] .'"';
}
```

Which I suggest the following change

```
if (isset($fieldMapping['options']['comment']) && $fieldMapping['options']['comment']) {
    $options[] = '"comment"="' . str_replace('"', '""', $fieldMapping['options']['comment']) .'"';
}
```

# Enjoy the result

```
bin/console d:m:import "App\Entity" annotation --path=src/Entity
Importing mapping information from "default" entity manager
  > writing src/Entity/Unicorn.php

bin/console make:entity --regenerate App

 updated: src/Entity/Unicorn.php
  Success!      
```