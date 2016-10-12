# 2. Recuperando dados por Objetos


>  AngularFire2 sincroniza dados como objetos usando o `FirebaseObjectObservable`. 
O `FirebaseObjectObservable` ele não se cria sozinho, mas sim atravéz do servico `AngularFire.database`. 
O guia abaixo demonstra como recuperar, salvar e remover dados por objetos.

## Injetando o servico AngularFire

**Certifique-se de ter configurado o AngularFire2 corretamente. Consulte o guia de instação e configuração.

AngularFire é um servico injetavel, ou seja que é injetado em nosso Componente ou serviços `@Injectable()`   através de seus construtores.

Se você seguiu o topico anterior de "Instalação e Configuração", seu arquivo  `/src/app/app.component.ts` deve se parecer com este abaixo.

```ts
import { Component } from '@angular/core';
import { AngularFire, FirebaseListObservable } from 'angularfire2';

@Component({
  selector: 'app-root',
  templateUrl: 'app.component.html',
  styleUrls: ['app.component.css']
})
export class AppComponent {
  items: FirebaseListObservable<any[]>;
  constructor(af: AngularFire) {
    this.items = af.database.list('items');
  }
}
```
Agora , vamos modificar o `/src/app/app.component.ts` para recuperar dados por objeto.

## Fazer "Binding" de um Objeto

Os dados são recuperados através do serviço `af.database`

Existem duas maneiras de se fazer "Binding" de objeto:
There are two ways to create an object binding:

1. Relative URL
1. Absolute URL

```ts
// relative URL, usa a url do banco de dados fornecido na inicialização
const relative = af.database.object('/item');
// absolute URL
const absolute = af.database.object('https://<your-app>.firebaseio.com/item');
```

### Recuperar dados 

Para se ter um objeto em tempo real, realizar "binding" de um objeto em seu component ou serviço, 
você deve, em seu template(HTML), usar pipe( | ) `async` ao realizar o "binding" do objeto.

No arquivo `/src/app/app.component.ts` substitua FirebaseListObservable por FirebaseObjectObservable.
Além disso, observe abaixo o template do componente que foi alterado.

```ts
import {Component} from '@angular/core';
import {AngularFire, FirebaseObjectObservable} from 'angularfire2';

@Component({
  selector: 'app-root',
  template: `
  <h1>{{ (item | async)?.name }}</h1>
  `,
})
export class AppComponent {
  item: FirebaseObjectObservable<any>;
  constructor(af: AngularFire) {
    this.item = af.database.object('/item');
  }
}
```

## Savar os dados

### Resumo API

A tabela a seguir destaca alguns dos métodos mais comuns que `FirebaseObjectObservable` ´possui.



| method   |                    | 
| ---------|--------------------| 
| set(value: any)      | Substitui o valor atual encontrado no bando de dados por um valor novo especificado por parametro. Isso é chamado de **atualização destrutiva**, exatamente porque exclui tudo que estava armazenado e salva o novo valor. | 
| update(value: Object)   | Atualiza o valor atual do banco de dados com o novo valor passado por parametro. Isso é chamado de ataulização ** não destrutiva**,porque somente atualiza o dados com o valor especificado. |
| remove()   | Exclui todos os dados presentes no laço. É o mesmo que invocar `set(null)`. |

## Retornando promises

Todos os métodos citados acima retornam uma promisse. 
No entanto raramente é nescessário usar a promisse para indicar um sucesso, 
pois o bando de dados já mantém em tempo real o objeto em sincronia.

A promise pode ser útil para capturar possível erros nas regras de seguraça ou para debugar.

```ts
const promise = af.database.object('/item').remove();
promise
  .then(_ => console.log('success'))
  .catch(err => console.log(err, 'You dont have access!'));
```

### Salvando dados

Utilize o método `set()` para realizar **atualizações destrutivas**

```ts
const itemObservable = af.database.object('/item');
itemObservable.set({ name: 'new name!'});
```

### Atualizando dados

Utilize o método `update()` para **atualizações não destrutivas**

```ts
const itemObservable = af.database.object('/item');
itemObservable.update({ age: newAge });
```

**Somente objetos não primitivos são permitidos para realizar atualizações**. Isto porque 
utilizar `.update()` em um primitivo é a mesma coisa que realizar um `.set()`.

### Deletando dados

Utilize o método `remove()` para remover dados do local do objeto

```ts
const itemObservable = af.database.object('/item');
itemObservable.remove();
```

**Exemplo app**: 

```ts
import { Component } from '@angular/core';
import { AngularFire, FirebaseObjectObservable } from 'angularfire2';

@Component({
  selector: 'app-root',
  template: `
  <h1>{{ item | async | json }}</h1>
  <input type="text" #newname placeholder="Name" />
  <input type="text" #newsize placeholder="Size" />
  <br />
  <button (click)="save(newname.value)">Set Name</button>
  <button (click)="update(newsize.value)">Update Size</button>
  <button (click)="delete()">Delete</button>
  `,
})
export class AppComponent {
  item: FirebaseObjectObservable<any>;
  constructor(af: AngularFire) {
    this.item = af.database.object('/item');
  }
  save(newName: string) {
    this.item.set({ name: newName });
  }
  update(newSize: string) {
    this.item.update({ size: newSize });
  }
  delete() {
    this.item.remove();
  }
}
```

## Meta-fields no objeto
Data retrieved from the object binding contains special properties retrieved from the unwrapped Firebase DataSnapshot.

| property |                    | 
| ---------|--------------------| 
| $key     | The key for each record. This is equivalent to each record's path in our database as it would be returned by `ref.key()`.|
| $value   | If the data for this child node is a primitive (number, string, or boolean), then the record itself will still be an object. The primitive value will be stored under `$value` and can be changed and saved like any other field.|


## Retrieving the snapshot
AngularFire2 unwraps the Firebase DataSnapshot by default, but you can get the data as the original snapshot by specifying the `preserveSnapshot` option. 

```ts
this.item = af.database.object('/item', { preserveSnapshot: true });
this.item.subscribe(snapshot => {
  console.log(snapshot.key)
  console.log(snapshot.val())
});
```

## Querying?

Because `FirebaseObjectObservable` synchronizes objects from the realtime database, sorting will have no effect for queries that are not also limited by a range. For example, when paginating you would provide a query with a sort and filter. Both the sort operation and the filter operation affect which subset of the data is returned by the query; however, because the resulting object is simply json, the sort order will not be preseved locally. Hence, for operations that require sorting, you are probably looking for a [list](3-retrieving-data-as-lists.md)

For filtering response data see [](4-querying-lists.md) and repeat 'Object' to yourself everytime you read the word 'List'. 

###[Next Step: Retrieving data as lists](3-retrieving-data-as-lists.md)
