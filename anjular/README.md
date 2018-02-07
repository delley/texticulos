# Angular: Notes
Miscellaneous notes on Angular

## LifeCycle Hooks

ngOnChanges()

ngOnInit()

ngDoCheck()

ngAfterContentInit()

ngAfterContentChecked()

ngAfterViewInit()

ngAfterViewChecked()

ngOnDestroy()

## Build options

Development environment

```bash
ng build
```

ou

```bash
ng build --dev
```

Production environment

```bash
ng build --prod
```

## Start server

Simple option

```bash
ng serve
```

Bind host and port

```bash
ng serve --host 192.168.0.54 --port 8080
```

## Miscellaneous

### Start fake API

```bash
json-server --host 192.168.0.54 banco-de-dados.json
```

### Fix Bootstrap error BrowserslistError: Unknown browser major at error

You can use Bootstrap 4, but first edit package.json in node_modules/bootstrap:

```bash
"browserslist": [
  "last 1 major version", <----- delete this line
  ">= 1%", <----- delete this line
  "Chrome >= 45",
  "Firefox >= 38",
  "Edge >= 12",
  "Explorer >= 10",
  "iOS >= 9",
  "Safari >= 9",
  "Android >= 4.4",
  "Opera >= 30"
],
```

