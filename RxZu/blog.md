# Introducing RxZu, an Engine for Intuitive Graphs

In the beginning, all was linear. 

We had in our hands an interface that allowed users to design conversations, entirely based on graphs. 
This was part of the Vonage AI studio, where anyone can create it's own intelligent virtual assistant. The kicker? It was entirely based on forms. 

But AI was the future, and forms, which our clients found unusable, most definitely were *not*. 

## The Search for a Graphs Library 

We realized we needed a visual approach to simplify the already complex world of conversation design. Something clever, snazzy, intuitive. 
And for that, we needed a graphs library that would satisfy some requirements:
1. Angular support
2. Lightweight
3. Extendable and customizable
4. Extensive support and community

And what do you know? Our search yielded zero results. 
The libraries we found were extremely heavy, and included outdated dependencies such as Lodash and Backbone. 
The options we looked into were not open-sourced and lacked a community. 
The implementations we found were outdated, lacking Typings, unfitting for Angular environment, and introduced endless complexity for the simplest use case.  

## Enter RxZu

So we created RxZu, named for Reactive Extensions ([RxJS](http://reactivex.io/)) and *Zu*, the Japanese word for illustration. 

RxZu is a diagrams engine system, built on top of RxJS, that takes the  graphic visualization to the next level in terms of performance, optimization, and customizability.

RxZu is composed of multiple parts: the core engine, which is in charge of handling the models and the synchronization between them, and the rendering engine which handles the rendering, and is based on the desired framework utilizing the core engine.

Some of the leading guidelines in the project are minimal, clean code and the ability for customizations and extendibility of the engine entities which are: 
* Nodes, the main building block of any graph, are the visual representation of data intersections.
* Ports, links got to start from some point.
* Links, symbols of connectivity and continuity.
* Labels, one might want to give a name to a link or even use it for links actions buttons
* Custom, want to add your entity? no problem.

#### Enough talking, let's code
**RxZu currently only implements Angular as a rendering engine therefor all code examples are for Angular.**

![Alt Text](https://github.com/Vonage/rxzu/raw/main/assets/draganddropexample.gif)
Let's start by creating a new Angular application, that will display a graph and have a drag n' drop interface to add more nodes.
```bash
ng new rxzu-angular
# wait for angular installation to finish
cd rxzu-angular
```
Install @rxzu/angular
```bash
npm i @rxzu/angular
```

Run the application
```bash
ng s
```
Let's enable production mode for `RxZu` in `main.ts`
```javascript
import { enableProdMode } from '@angular/core';
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppModule } from './app/app.module';
import { environment } from './environments/environment';
import { enableDiagramProdMode } from '@rxzu/angular';

if (environment.production) {
  enableProdMode();
  enableDiagramProdMode();
}

platformBrowserDynamic()
  .bootstrapModule(AppModule)
  .catch((err) => console.error(err));
```

Add to `app.module.ts` RxZu module along with all the default component.
```javascript
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import {
  ComponentProviderOptions,
  DefaultLabelComponent,
  DefaultLinkComponent,
  DefaultNodeComponent,
  DefaultPortComponent,
  RxZuModule,
} from '@rxzu/angular';

import { AppComponent } from './app.component';

const DEFAULTS: ComponentProviderOptions[] = [
  {
    type: 'node',
    component: DefaultNodeComponent,
  },
  {
    type: 'port',
    component: DefaultPortComponent,
  },
  {
    type: 'link',
    component: DefaultLinkComponent,
  },
  {
    type: 'label',
    component: DefaultLabelComponent,
  },
];

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, CommonModule, RxZuModule.withComponents(DEFAULTS)],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

RxZu module `withComponents` method accepts an array of components and their type, this way the library can resolve and paint the different components when added to the diagram model which we'll create later in the tutorial.

Now let's create a cool stylish grid as our background, the draggable nodes and our action bar container
`app.component.scss`
```scss
.demo-diagram {
  display: flex;
  height: 100%;
  min-height: 100vh;
  background-color: #3c3c3c;
  background-image: linear-gradient(
      0deg,
      transparent 24%,
      rgba(255, 255, 255, 0.05) 25%,
      rgba(255, 255, 255, 0.05) 26%,
      transparent 27%,
      transparent 74%,
      rgba(255, 255, 255, 0.05) 75%,
      rgba(255, 255, 255, 0.05) 76%,
      transparent 77%,
      transparent
    ),
    linear-gradient(
      90deg,
      transparent 24%,
      rgba(255, 255, 255, 0.05) 25%,
      rgba(255, 255, 255, 0.05) 26%,
      transparent 27%,
      transparent 74%,
      rgba(255, 255, 255, 0.05) 75%,
      rgba(255, 255, 255, 0.05) 76%,
      transparent 77%,
      transparent
    );
  background-size: 50px 50px;
}

.node-drag {
  display: block;
  cursor: grab;
  background-color: white;
  border-radius: 30px;
  padding: 5px 15px;
}

.action-bar {
  position: fixed;
  width: 100%;
  height: 40px;
  z-index: 2000;
  background-color: rgba(255, 255, 255, 0.4);
  display: flex;
  align-items: center;

  * {
    margin: 0 10px;
  }
}

```

Our html template with the actions bar and the diagram itself.
`app.component.html`
```html
<div class="action-bar">
  <div
    *ngFor="let node of nodesLibrary"
    class="node-drag"
    draggable="true"
    [attr.data-name]="node.name"
    (dragstart)="onBlockDrag($event)"
    [ngStyle]="{ 'background-color': node.color }"
  >
    {{ node.name }}
  </div>
</div>

<rxzu-diagram
  class="demo-diagram"
  [model]="diagramModel"
  (drop)="onBlockDropped($event)"
  (dragover)="$event.preventDefault()"
></rxzu-diagram>
```

And for the last piece in the puzzle, create some nodes, ports, link them up and render it all.
`app.component.ts`
```js
import { AfterViewInit, Component, ViewChild } from '@angular/core';
import {
  DiagramModel,
  NodeModel,
  PortModel,
  RxZuDiagramComponent,
} from '@rxzu/angular';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss'],
})
export class AppComponent implements AfterViewInit {
  diagramModel: DiagramModel;
  nodesLibrary = [
    { color: '#AFF8D8', name: 'default' },
    { color: '#FFB5E8', name: 'default' },
    { color: '#85E3FF', name: 'default' },
  ];
  @ViewChild(RxZuDiagramComponent, { static: true })
  diagram?: RxZuDiagramComponent;

  constructor() {
    this.diagramModel = new DiagramModel();
  }

  ngAfterViewInit() {
    this.diagram?.zoomToFit();
  }

  createNode(type: string) {
    const nodeData = this.nodesLibrary.find((nodeLib) => nodeLib.name === type);
    if (nodeData) {
      const node = new NodeModel();
      const port = new PortModel();
      node.addPort(port);
      node.setExtras(nodeData);

      return node;
    }

    return null;
  }

  /**
   * On drag start, assign the desired properties to the dataTransfer
   */
  onBlockDrag(e: DragEvent) {
    const type = (e.target as HTMLElement).getAttribute('data-type');
    if (e.dataTransfer && type) {
      e.dataTransfer.setData('type', type);
    }
  }

  /**
   * on block dropped, create new intent with the empty data of the selected block type
   */
  onBlockDropped(e: DragEvent): void | undefined {
    if (e.dataTransfer) {
      const nodeType = e.dataTransfer.getData('type');
      const node = this.createNode(nodeType);
      const canvasManager = this.diagram?.diagramEngine.getCanvasManager();
      if (canvasManager) {
        const droppedPoint = canvasManager.getZoomAwareRelativePoint(e);
        const width = node?.getWidth() ?? 1;
        const height = node?.getHeight() ?? 1;
        const coords = {
          x: droppedPoint.x - width / 2,
          y: droppedPoint.y - height / 2,
        };

        if (node) {
          node.setCoords(coords);
          this.diagramModel.addNode(node);
        }
      }
    }
  }
}
```
We want to believe the code is self explanatory, but i'll do a quick overview nevertheless.

the `diagramModel` is our most important part, it hold the entire diagram model (duh), and in order to add or remove elements from the diagram we work on the model.

```javascript
this.diagramModel.addNode(node);
```

some entities are children of others, such as ports which are the children nodes.
and can be added by directly attaching them to their parent.
```javascript
const port = new PortModel();
node.addPort(port);
```
In the next tutorial I'll show how to create customized nodes, that utilize the extra information passed to them.

Till then you can find many more examples at our [Storybook](https://vonage.github.io/rxzu), and the source code at our [GitHub](https://github.com/Vonage/rxzu)

#### What the future holds for us?
One of the most important tasks we have ahead is getting better performance in the core.
Adding support for React, Vue, and more...
Smarter links with obstacles awareness.
Rendering only elements in view port to support gigantic diagrams (thousands of entities)
And much more....
