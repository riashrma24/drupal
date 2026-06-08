Twig is a templating engine for PHP.
BE prepares data, FE prepares presentation, Twig acts as a bridge

Reasons to not directly use php in html pages,
1. HTML becomes unreadable
2. Templates should only render UI, they shouldnt Query database, Delete users, Process payments. Twig intentionally restricts capabilities.
3. XSS attack possible. XSS(Cross site scripting attack) : It happens when an attacker manages to inject JavaScript into a webpage that other users will view.
If an attacker enters <script>alert("Hacked!")</script>, db stores <script>alert("Hacked!")</script>


Twig architecture : 
1. Data stored as
$data = [
    "name" => "Riya",
    "age" => 23
];
2. Controller

return $twig->render(
    "profile.twig",
    $data
);

3. Twig

d<h1>{{ name }}</h1>
d<p>{{ age }}</p>

4. HTML

<h1>Riya</h1>
<p>23</p>

### How twig works internally?

1. .twig files are compiled into php. (Twig is not interpreted at every request)

### Twig only has 3 major delimiters :
1. Print/Output {{ }}
2. Logic {% %}
3. Comments {# #}

### Accessing objects : 
1. In php : $user->name
2. In twig : {{ user.name }}
Twig internally tries $user->name or $user->getName() or $user['name'] depending on object type

### Accessing arrays :
1. {{ skills[0] }}

### Two types of loops in twig : 
1. Normal loop 

{% for skill in skills %}
   {{ skill }}
{% endfor %}

2. Key Value Loops

{% for key,value in user %}
    {{ key }} : {{ value }}
{% endfor %}

Twig also creates some loop variables : 
loop.index
loop.index0
loop.first
loop.last
loop.length

### Filters in twig

1. {{ name|upper }} -> upper
2. {{ name|lower }} -> lower
3. {{ skills|length }} -> length
4. {{ text|trim }} -> trim
5. {{ text|replace({'hello':'hi'}) }} -> replace
6. {{ skills|join(', ') }} -> join
7. {{ username|default('Guest') }} -> default

### Functions
1. {{ range(1,5) }}
2. 


### Set Variable

{% set name = "Riya" %}

### Include, extends, 

1. extends
   - creates a parent child relationship between templates.
   - only one parent possible, multiple parents not allowed.
   - Used for : Page layouts, Site layouts, Master templates

eg : in java

class Animal {
    void eat() {}
}

class Dog extends Animal {
}

Twig:

{% extends "base.html.twig" %}

eg : Parent

{# base.html.twig #}

<html>
<head>
    {% block head %}
    {% endblock %}
</head>

<body>
    {% block content %}
    {% endblock %}
</body>
</html>

child

{% extends "base.html.twig" %}

{% block content %}
    <h1>Profile Page</h1>
{% endblock %}

generated html

<html>
<head>
</head>

<body>
    <h1>Profile Page</h1>
</body>
</html>

2. include
   - simply inserts another template.
   - No blocks overridden.
   - No parent-child relationship.
   - Just rendering another template.
   - used for : Header, Footer, Sidebar, Card component, Reusable snippets
  
3. embed
   - embed = include + extends
   - 



In drupal,
Browser req -> Drupal Core -> Theme Layer -> Preprocess Hooks -> Twig Template -> HTML Output
