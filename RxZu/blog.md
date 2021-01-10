## All we need is graph, graph is all we need
Introducing RxZu, Vonage graphs engine.

#### First thing first, what is RxZu?
RxZu is a combination of Rx which is a shorthand for Reactive Extensions and in our specific case Reactive Extensions for Javascript also known as RxJS.
Zu is the Japanese word for drawing, picture, diagram, figure, illustration, chart, graph.
That leads us to an understanding that RxZu is a diagrams engine system, build on top of RxJS, which takes the idea of graphic visualization to the next level in terms of performance, optimization, and customizability.

RxZu is composed of multiple parts, the core engine which is in charge of handling the models and the synchronization between them, and the rendering engine which is in charge of the, well, the rendering of course, and is based on the desired framework utilizing the core engine.

Some of the leading guidelines in the project are minimal, clean code and the ability for customizations and extendibility of the engine entities which are: 
* Nodes, the main building block of any graph, are the visual representation of data intersections.
* Ports, links got to start from some point.
* Links, symbols of connectivity and continuity.
* Labels, one might want to give a name to a link or even use it for links actions buttons
* Custom, want to add your own entity? no problem.

#### Enough talking, let's code
**RxZu currently only implements Angular as a rendering engine therefor all code examples are for Angular.**
Let's start by creating a new Angular application
```bash
ng new rxzu-angular
# wait for angular installation to finish
cd rxzu-angular
```
Install @rxzu/angular
```bash
npm i @rxzu/angular
```
Navigate to `./tsconfig.json` and change `"strict": true` to `"strict": false`, sadly we don't yet support this and it will introduce some generics typings issues. 
**rest assured this is a work in progress.**

Run the application
```bash
ng s
```

Add to `app.module.ts` RxZu module
```javascript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';
import { RxZuDiagramsModule } from '@rxzu/angular';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, RxZuDiagramsModule],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

Now let's create a cool stylish grid as our background
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
```

And for the last piece in the puzzle, create some nodes, ports, link them up and render it all.
`app.component.ts`
```js
import { Component, OnInit } from '@angular/core';
import { DiagramEngine } from '@rxzu/angular';
import { DiagramModel, DefaultNodeModel } from '@rxzu/core';

@Component({
  selector: 'app-root',
  template: `<ngdx-diagram
    class="demo-diagram"
    [inverseZoom]="true"
    [model]="diagramModel"
  ></ngdx-diagram>`,
  styleUrls: ['./app.component.scss'],
})
export class AppComponent implements OnInit {
  diagramModel: DiagramModel;

  constructor(private diagramEngine: DiagramEngine) {}

  ngOnInit() {
    const nodesDefaultDimensions = { height: 200, width: 200 };
    this.diagramEngine.registerDefaultFactories();

    this.diagramModel = this.diagramEngine.createModel();

    const node1 = new DefaultNodeModel();
    node1.setCoords({ x: 500, y: 300 });
    node1.setDimensions(nodesDefaultDimensions);
    const outport1 = node1.addOutPort({ name: 'outport1' });

    const node2 = new DefaultNodeModel();
    node2.setCoords({ x: 1000, y: 0 });
    node2.setDimensions(nodesDefaultDimensions);
    const inport = node2.addInPort({ name: 'inport2' });

    const link = outport1.link(inport);
    link.setLocked();

    this.diagramModel.addAll(node1, node2, link);

    this.diagramModel.getDiagramEngine().zoomToFit();
  }
}
```
We want to believe the code is self explanatory, but i'll do a quick overview nevertheless.

```js
this.diagramEngine.registerDefaultFactories();
```
As the name states, registers all default factories provided out of the box by RxZu as a starting point, their source code can be found at https://github.com/Vonage/rxzu/tree/main/packages/angular/src/lib/defaults/components and it is highly recommended to overview it when moving forward into fully customized entities.

```js
const node1 = new DefaultNodeModel();
node1.setCoords({ x: 500, y: 300 });
node1.setDimensions(nodesDefaultDimensions);
const outport1 = node1.addOutPort({ name: 'outport1' });
```
Instantiating a node entity, which in turn generates the node component and exposes the model for us to manipulate it, update the coordinates, change the dimensions, create outport that is also an entity that instantiates behind the scenes and have lots of manipulations of their own.

I'll stop here, there's plenty more to do and show using RxZu and this is probably the first of many posts about RxZu.

You can find the source code at our [GitHub](https://github.com/Vonage/rxzu), and read the doc’s and stories at our [Storybook](https://vonage.github.io/rxzu)


#### What the future holds for us?
One of the most important tasks we have ahead is getting better performance in the core.
Adding support for React, Vue, and more...
Smarter links with obstacles awareness.
Improving the documentation.
