# 📝 Jinja2 Syntax Cheat Sheet

## 1. **Basic Expressions**
```jinja2
{{ variable }}          # Output a variable
{{ user.name }}         # Access object attribute
{{ user["name"] }}      # Access dictionary key
{{ 3 + 2 }}             # Arithmetic → 5
{{ "Hello"|upper }}     # Filters → HELLO
```

---

## 2. **Control Structures**

### **If / Else**
```jinja2
{% if user %}
  Hello, {{ user.name }}
{% else %}
  Hello, Guest
{% endif %}
```

### **For Loops**
```jinja2
{% for item in items %}
  {{ loop.index }} - {{ item }}
{% endfor %}

# loop helpers:
loop.index      # 1-based index
loop.index0     # 0-based index
loop.revindex   # Reverse index
loop.first      # True on first iteration
loop.last       # True on last iteration
```

---

## 3. **Filters**
```jinja2
{{ "hello world"|title }}     # Hello World
{{ "HELLO"|lower }}           # hello
{{ list|length }}             # Count items
{{ items|join(", ") }}        # Join list
{{ price|float|round(2) }}    # Rounds number
```

---

## 4. **Macros (Reusable Functions)**
```jinja2
{% macro render_input(name, value='') %}
  <input type="text" name="{{ name }}" value="{{ value }}">
{% endmacro %}

{{ render_input("username") }}
```

---

## 5. **Includes & Extends**

### **Include partial template**
```jinja2
{% include "header.html" %}
```

### **Template Inheritance**
```jinja2
# base.html
<html>
  <body>
    {% block content %}{% endblock %}
  </body>
</html>

# child.html
{% extends "base.html" %}
{% block content %}
  <h1>Welcome</h1>
{% endblock %}
```

---

## 6. **Whitespace Control**
```jinja2
{{- variable -}}   # Trim whitespace around output
{% -%} ... {%- %}  # Trim whitespace around blocks
```

---

## 7. **Set Variables**
```jinja2
{% set total = price * qty %}
Total: {{ total }}
```

---

## 8. **Comments**
```jinja2
{# This is a comment #}
```

---

## 9. **Filters & Tests**

### **Common Tests**
```jinja2
{% if value is none %}    # None check
{% if value is string %}
{% if value is number %}
{% if list is iterable %}
{% if list is defined %}
```
