---
title: test functions
menu:
  main:
    parent: functions
---

The `test` namespace contains some simple functions to help validate
assumptions and can cause template generation to fail in specific cases.

## `test.Assert`

**Alias:** `assert`

Asserts that the given expression or value is `true`. If it is not, causes
template generation to fail immediately with an optional message.

### Usage
```go
test.Assert [message] value 
```

```go
value | test.Assert [message]  
```

### Arguments

| name | description |
|------|-------------|
| `message` | _(optional)_ The optional message to provide in the case of failure |
| `value` | _(required)_ The value to test |

### Examples

```console
$ gomplate -i '{{ assert (eq "foo" "bar") }}'
template: <arg>:1:3: executing "<arg>" at <assert (eq "foo" "ba...>: error calling assert: assertion failed
$ gomplate -i '{{ assert "something horrible happened" false }}'
template: <arg>:1:3: executing "<arg>" at <assert "something ho...>: error calling assert: assertion failed: something horrible happened
```

## `test.Fail`

**Alias:** `fail`

Cause template generation to fail immediately, with an optional message.

### Usage
```go
test.Fail [message] 
```

```go
message | test.Fail  
```

### Arguments

| name | description |
|------|-------------|
| `message` | _(optional)_ The optional message to provide |

### Examples

```console
$ gomplate -i '{{ fail }}'
template: <arg>:1:3: executing "<arg>" at <fail>: error calling fail: template generation failed
$ gomplate -i '{{ test.Fail "something is wrong!" }}'
template: <arg>:1:7: executing "<arg>" at <test.Fail>: error calling Fail: template generation failed: something is wrong!
```

## `test.Required`

**Alias:** `required`

Passes through the given value, if it's non-empty, non-zero, and non-`nil`.
Otherwise, exits and prints a given error message so the user can adjust as
necessary.

This is particularly useful for cases where templates require user-provided
data (such as datasources or environment variables), and rendering can not
continue correctly.

This was inspired by [Helm's `required` function](https://github.com/kubernetes/helm/blob/master/docs/charts_tips_and_tricks.md#know-your-template-functions),
but has slightly different behaviour. Notably, gomplate will always fail in
cases where a referenced _key_ is missing, and this function will have no
effect.

### Usage
```go
test.Required [message] value 
```

```go
value | test.Required [message]  
```

### Arguments

| name | description |
|------|-------------|
| `message` | _(optional)_ The optional message to provide when the required value is not provided |
| `value` | _(required)_ The required value |

### Examples

```console
$ FOO=foobar gomplate -i '{{ getenv "FOO" | required "Missing FOO environment variable!" }}'
foobar
$ FOO= gomplate -i '{{ getenv "FOO" | required "Missing FOO environment variable!" }}'
error: Missing FOO environment variable!
```
```console
$ cat <<EOF> config.yaml
defined: a value
empty: ""
EOF
$ gomplate -d config=config.yaml -i '{{ (ds "config").defined | required "The `config` datasource must have a value defined for `defined`" }}'
a value
$ gomplate -d config=config.yaml -i '{{ (ds "config").empty | required "The `config` datasource must have a value defined for `empty`" }}'
template: <arg>:1:25: executing "<arg>" at <required "The `confi...>: error calling required: The `config` datasource must have a value defined for `empty`
$ gomplate -d config=config.yaml -i '{{ (ds "config").bogus | required "The `config` datasource must have a value defined for `bogus`" }}'
template: <arg>:1:7: executing "<arg>" at <"config">: map has no entry for key "bogus"
```
