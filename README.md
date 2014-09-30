#PHP Matcher

***PHP Matcher*** lets You assert like a gangster in Your test cases, where response can be something you cannot predict

[![Build Status](https://travis-ci.org/coduo/php-matcher.svg)](https://travis-ci.org/coduo/php-matcher)

##Installation

Add to your composer.json 

```
require: {
   "coduo/php-matcher": "1.0.*"
}
```

### Ways of testing
Common way of testing api responses with Symfony2 WebTestCase

```php
public function testGetToys()
{
    $this->setUpFixtures();
    $this->setUpOath();
    $this->client->request('GET', '/api/toys');
    $response = $this->client->getResponse();
    $this->assertJsonResponse($response, 200);
    $content = $response->getContent();
    $decoded = json_decode($content, true);
    $this->assertTrue(isset($decoded[0]['id']));
    $this->assertTrue(isset($decoded[1]['id']));
    $this->assertTrue(isset($decoded[2]['id']));
    $this->assertEquals($decoded[0]['name'], 'Barbie'));
    $this->assertEquals($decoded[1]['name'], 'GI Joe'));
    $this->assertEquals($decoded[2]['name'], 'Optimus Prime'));
}
```
With php-matcher, you can make it more readable:
```php
public function testGetToys()
{
    $this->setUpFixtures();
    $this->setUpOath();
    $this->client->request('GET', '/api/toys');
    $response = $this->client->getResponse();
    $this->assertJsonResponse($response, 200);
    $content = $response->getContent();
    $pattern = '[
        {
          "id": @string@,
          "name": "Barbie",
          "_links: @*@
        },
        {
          "id": @string@,
          "name": "GI Joe",
          "_links": @*@
        },
        {
          "id": @string@,
          "name": "Optimus Prime",
          "_links": @*@
        }
      ]
   ';
   $this->assertTrue(match($content, $pattern));
}
```


From now you should be able to use global function ``match($value, $pattern)``

### Available patterns

* ``@string@``
* ``@integer@``
* ``@boolean@``
* ``@array@``
* ``@...@`` - *unbounded array*
* ``@double@``
* ``@null@``
* ``@*@`` || ``@wildcard@``
* ``expr(expression)``

##Example usage

### Scalar matching

```php
<?php

match(1, 1);
match('string', 'string')
```

### Type matching

```php
<?php

match(1, '@integer@');
match('Norbert', '@string@');
match(array('foo', 'bar'), '@array');
match(12.4, '@double@');
match(true, '@boolean@');
```

### Wildcard 

```php
<?php

match(1, '@*@');
match(new \stdClass(), '@wildcard@');
```

### Expression matching 

```php
<?php

match(new \DateTime('2014-04-01'), "expr(value.format('Y-m-d') == '2014-04-01'");
match("Norbert", "expr(value === 'Norbert')");
```

### Array matching 

```php
<?php

match(
   array(
      'users' => array(
          array(
              'id' => 1,
              'firstName' => 'Norbert',
              'lastName' => 'Orzechowicz',
              'roles' => array('ROLE_USER')
          ),
          array(
              'id' => 2,
              'firstName' => 'Michał',
              'lastName' => 'Dąbrowski',
              'roles' => array('ROLE_USER')
          ),
          array(
              'id' => 3,
              'firstName' => 'Johnny',
              'lastName' => 'DąbrowsBravoki',
              'roles' => array('ROLE_HANDSOME_GUY')
          )
      ),
      true,
      6.66
  ),
   array(
      'users' => array(
          array(
              'id' => '@integer@',
              'firstName' => '@string@',
              'lastName' => 'Orzechowicz',
              'roles' => '@array@'
          ),
          array(
              'id' => '@integer@'
              'firstName' => '@string@',
              'lastName' => 'Dąbrowski',
              'roles' => '@array@'
          ),
          '@...@'
      ),
      '@boolean@',
      '@double@'
  )  
)
```

### Json matching 


```php
<?php

match(
  '{
    "users":[
      {
        "firstName": "Norbert",
        "lastName": "Orzechowicz",
        "roles":["ROLE_USER", "ROLE_DEVELOPER"]}
      ]
  }',
  '{
    "users":[
      {
        "firstName": @string@,
        "lastName": @string@,
        "roles": @array@
      }
    ]
  }'
)

```

### Xml matching


```php
<?php

match(
    <<<XML
<?xml version="1.0"?>
<soap:Envelope
xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">

<soap:Body xmlns:m="http://www.example.org/stock">
  <m:GetStockPrice>
    <m:StockName>IBM</m:StockName>
    <m:StockValue>Any Value</m:StockValue>
  </m:GetStockPrice>
</soap:Body>

</soap:Envelope>
XML
    ,
   <<<XML
<?xml version="1.0"?>
<soap:Envelope
    xmlns:soap="@string@"
            soap:encodingStyle="@string@">

<soap:Body xmlns:m="@string@">
  <m:GetStockPrice>
    <m:StockName>@string@</m:StockName>
    <m:StockValue>@string@</m:StockValue>
  </m:GetStockPrice>
</soap:Body>

</soap:Envelope>
XML
)

```

Example scenario for api in behat using mongo.
---
``` cucumber
@profile, @user
Feature: Listing user toys

  As a user
  I want to list my toys

  Background:
    Given I send and accept JSON

  Scenario: Listing toys
    Given the following users exist:
      | firstName     | lastName     |
      | Chuck         | Norris       | 

    And the following toys user "Chuck Norris" exist:
      | name            |
      | Barbie          |
      | GI Joe          |
      | Optimus Prime   |

    When I set valid authorization code oauth header for user "Chuck Norris"
    And I send a GET request on "/api/toys"
    Then the response status code should be 200
    And the JSON response should match:
    """
      [
        {
          "id": @string@,
          "name": "Barbie",
          "_links: "@*@"
        },
        {
          "id": @string@,
          "name": "GI Joe",
          "_links": "@*@"
        },
        {
          "id": @string@,
          "name": "Optimus Prime",
          "_links": "@*@"
        }
      ]
    """
``` 

## License

This library is distributed under the MIT license. Please see the LICENSE file.

## Credits

This lib was inspired by [JSON Expressions gem](https://github.com/chancancode/json_expressions) && [Behat RestExtension ](https://github.com/jakzal/RestExtension)

