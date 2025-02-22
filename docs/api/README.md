# API

## `registerMicroApps(apps, lifeCycles?)`

- Parameters

  - apps - `Array<RegistrableApp<T>>` - required, registration information for the child application
  - lifeCycles - `LifeCycles<T>` - optional, global sub app lifecycle hooks

- Type

  - `RegistrableApp<T>`

    - name - `string` - required, the name of the child application and must be unique between the child applications.

    - entry - `string | { scripts?: string[]; styles?: string[]; html?: string }` - required, The entry url of the child application.

    - container - `string | HTMLElement` - required，A selector or Element instance of the container node of a micro application. Such as `container: '#root'` or `container: document.querySelector('#root')`。

    - activeRule -  - `string | (location: Location) => boolean | Array<string | (location: Location) => boolean> ` - required.

      This function is called when the browser url changes, and `activeRule` returns `true` to indicate that the subapplication needs to be activated.

    - props - `object` - optional, data that the primary application needs to pass to the child application.

  - `LifeCycles<T>`

    ```ts
    type Lifecycle<T extends object> = (app: RegistrableApp<T>) => Promise<any>;
    ```

    - beforeLoad - `Lifecycle<T> | Array<Lifecycle<T>>` - optional
    - beforeMount - `Lifecycle<T> | Array<Lifecycle<T>>` - optional
    - afterMount - `Lifecycle<T> | Array<Lifecycle<T>>` - optional
    - beforeUnmount - `Lifecycle<T> | Array<Lifecycle<T>>` - optional
    - afterUnmount - `Lifecycle<T> | Array<Lifecycle<T>>` - optional

- Usage

  Configuration information for registered subapplications in the main application.

- Sample

  ```tsx
  import { registerMicroApps } from 'qiankun';

  registerMicroApps(
    [
      {
        name: 'app1',
        entry: '//localhost:8080',
        container: '#container',
        activeRule: '/react',
        props: {
          name: 'kuitos',
        }
      }
    ],
    {
      beforeLoad: app => console.log('before load', app.name),
      beforeMount: [
        app => console.log('before mount', app.name),
      ],
    },
  );
  ```

## `start(opts?)`

- Parameters

  - opts - `Options` optional

- Type

  - `Options`

    - prefetch - `boolean | 'all' | string[] | (( apps: RegistrableApp[] ) => { criticalAppNames: string[]; minorAppsName: string[] })` - optional, whether to enable prefetch, default is `true`.

      A configuration of `true` starts prefetching static resources for other subapplications after the first subapplication mount completes.
  
      If configured as `'all'`, the main application `start` will begin to preload all subapplication static resources.

      If configured as `string[]`, starts prefetching static resources for subapplications after the first subapplication mount completes which be declared in this list.

      If configured as `function`, the timing of all subapplication static resources will be controlled by yourself.

    - sandbox - `boolean` | `{ strictStyleIsolation?: boolean }` - optional, whether to open the js sandbox, default is `true`.
      
      When configured as `{strictStyleIsolation: true}`, qiankun will convert the container dom of each application to a [shadow dom](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM), to ensure that the style of the application will not leak to the global.

    - singular - `boolean | ((app: RegistrableApp<any>) => Promise<boolean>);` - optional，whether is a singular mode，default is `true`.

    - fetch - `Function` - optional
    
    - getPublicPath - `(url: string) => string` - optional
    
    - getTemplate - `(tpl: string) => string` - optional

- Usage

  Start qiankun.

- Sample

  ```ts
  import { start } from 'qiankun';

  start();
  ```

## `setDefaultMountApp(appLink)`

- Parameters

  - appLink - `string` - required

- Usage

  Sets the child application that enters by default after the main application starts.

- Sample

  ```ts
  import { setDefaultMountApp } from 'qiankun';

  setDefaultMountApp('/homeApp');
  ```

## `runAfterFirstMounted(effect)`

- Parameters

  - effect - `() => void` - required

- Usage

  Methods that need to be called after the first subapplication mount, such as turning on some monitoring or buried scripts.

- Sample

  ```ts
  import { runAfterFirstMounted } from 'qiankun';

  runAfterFirstMounted(() => startMonitor());
  ```

## [addErrorHandler/removeErrorHandler](https://single-spa.js.org/docs/api#adderrorhandler)

## `addGlobalUncaughtErrorHandler(handler)`

- Parameters

  - handler - `(...args: any[]) => void` - 必选

- Usage

  Add the global uncaught error hander.

- Sample

  ```ts
  import { addGlobalUncaughtErrorHandler } from 'qiankun';
  
  addGlobalUncaughtErrorHandler(event => console.log(event));
  ```

## `removeGlobalUncaughtErrorHandler(handler)`

- Parameters

  - handler - `(...args: any[]) => void` - 必选

- Usage

  Remove the global uncaught error hander.

- Sample

  ```ts
  import { removeGlobalUncaughtErrorHandler } from 'qiankun';
  
  removeGlobalUncaughtErrorHandler(handler);
  ```

## `initGloabalState(state)`

- Parameters

  - state - `Record<string, any>` - 必选

- Usage

  init global state, and return actions for communication. It is recommended to use in master, and slave get actions through props。

- Return

  - MicroAppStateActions

    - onGlobalStateChange: `(callback: OnGlobalStateChangeCallback, fireImmediately?: boolean) => void` - Listen the global status in the current application: when state changes will trigger callback; fireImmediately = true, will trigger callback immediately when use this method.

    - setGlobalState: `(state: Record<string, any>) => boolean` - Set global state by first layer props, it can just modify first layer props what has defined.

    - offGlobalStateChange: `() => boolean` - Remove Listener in this app, will default trigger when app unmount.

- Sample

  Master:
  ```ts
  import { initGloabalState, MicroAppStateActions } from 'qiankun';

  const actions: MicroAppStateActions = initGloabalState(state);

  actions.onGlobalStateChange((state, prev) => {
    // state: new state; prev old state
    console.log(state, prev);
  });
  actions.setGlobalState(state);
  actions.offGlobalStateChange();
  ```

  Slave:
  ```ts
  // get actions from mount
  export function mount(props) {

    props.onGlobalStateChange((state, prev) => {
      // state: new state; prev old state
      console.log(state, prev);
    });
    props.setGlobalState(state);

    // It will trigger when slave umount,  not necessary to use in non special cases.
    props.offGlobalStateChange();
    
    // ...
  }
  ```
