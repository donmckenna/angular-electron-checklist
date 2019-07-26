# Angular Inside Electron Checklist

### Motivation
Made this as a checklist for myself to translate sleep-deprived ramblings scribbled in coffee-stained notebooks into something readable and fireproof.


### Disclaimer
I take no responsibility if anyone uses this checklist and subsequently is hit by a falling piano. These settings and directories work for my use but there is no guarantee they will work for yours. This is by no means meant to be a comprehensive tutorial, but I hope it can at least bring some pieces together and help someone else not age more rapidly than absolutely necessary.

<br>
<hr>

### How to watch (Webpack hot reload) an Angular app with routing inside an Electron app and refresh them both on an Angular `src/app/*` file change: 

1. `yarn add -D electron-reload`

2.  At the top of `main.js`:
    ```
    require('electron-reload')(__dirname, {
      electron: require(`${__dirname}/node_modules/electron`)
    });
    ```

3. In `index.html` between the `<head>` tags:  
  `<base href="./index.html">`
      - **Needs the full `./index.html`**
      - **NOT** `/`
      - **NOT** `.`
      - **NOT** `./`

4. In `app-routing-module.ts`  
    - Electron conflicts with Angular's routing if Angular's routing does not use hash.
    ```
    @NgModule({
      imports: [RouterModule.forRoot(routes, { useHash: true })],
      ..
    })
    ```

5. Open 2 terminal windows and `cd` into your project's root directory.  
    - In the first:  
    `ng build --watch`
    - In the second:  
    `electron .`

<br>
### NOTES

If you're more familiar with Electron than Angular and wondering where `require("../renderer.js")` goes, the answer is "nowhere".

Angular _**IS**_ the renderer.

This is why `require("../renderer.js")` doesn't do anything with this setup.

`ipcRenderer` is included directly into Angular when imported into a component. Angular itself is listening for the main process changes. It doesn't need another intermediary between it and the main process.

For example, minimally:
```
import { ipcRenderer } from 'electron';

constructor() {
  ipcRenderer.on('some-action', this.someMethod.bind(this));
}

someMethod() {
  console.log('hi');
}
```
..or another way:
```
import { NgZone } from '@angular/core';
import { ipcRenderer } from 'electron';

constructor( private zone: NgZone ) {
  ipcRenderer.on('some-action', () => {
    this.zone.run(() => this.someMethod() );
  });
}

someMethod() {
  console.log('hi');
}

```


