# Broken access control

This vulnerability occurs when the `Shiro` and `Spring` frameworks are used together and the `Spring` framework uses the `Shiro` framework to identify user permissions. Because the `Shiro` and `Spring` frameworks handle the request URL differently, the user can access unauthorized pages by constructed specific URL.

## Background

### Filter order

First `Tomcat filter` then `Shiro filter`.

### Shiro filter pattern

* `anon`: Anonymous filter, can access the page without login.

* `authc`: Authentication filter, must login and be authenticated to access the page.

### Shiro filter match pattern

Match the URL path for the access route.

* `?` : Match a character.

* `*` : Match a or more characters.

* `**` : Match a or more directories.

## 1 CVE-2016-6802

### 1.1 Condition

`Shiro` version `< 1.5.0`.

`Shiro` matches `"/admin"`.

### 1.2 Analysis

Request URL: `"/admin/"`.

`Shiro` treatment: Compare the request `"/admin/"` URL with the `"/admin"` pattern.

`Spring` treatment: Add `'/'` to the end of pattern, then compare.

### 1.3 Patch

Delete the `'/'` at the end of the request URL.

## 2 CVE-2020-1957

### 2.1 Condition

`Shiro` version `< 1.5.2`.

### 2.2 Analysis

Request URL: `"/;/admin/"`.

`Shiro` treatment: Delete the characters after `';'`.

`Spring` treatment: Delete the characters after `';'` and before `'/'`.

### 2.3 Patch

Use `request.getContextPath()`, `request.getServletPath()`, `request.getPathInfo()` methods to concatenate URL, instead `request.getRequestURI()` method. And the `request.getServletPath()` method handle `';'` correctly.

## 3 CVE-2020-11989

### 3.1 Condition

`Shiro` version `< 1.5.3`.

`Shiro` matches `"/admin/*"`, `Spring` matches `"/admin/{String}"`

### 3.2 Analysis

Request URL: `"/admin/index%25%32%66test"`.

`Shiro` treatment: Decode the URL which is decoded to `"/admin/index%2ftest"` by the `Tomcat` server. The URL becomes `"/admin/index/test"` and doesn't match the pattern.

`Spring` treatment: The request URL will not be decoded twice.

### 3.3 Patch

Delete the URL decode function.

## 4 CVE-2020-13933

### 4.1 Condition

`Shiro` version `< 1.6.0`.

`Shiro` matches `"/admin/*"`, `Spring` matches `"/admin/{String}"`

### 4.2 Analysis

Request URL: `"/admin/%3b"`.

`Shiro` treatment: Obtain the decoded URL, then Delete the characters after `';'`. And the URL `/admin/` doesn't match the pattern.

`Spring` treatment: Handle the `';'`, then decode the request URL.

### 4.3 Patch

Ban some specail characters in the request URL.
