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

## Meta-fields do objeto

Os dados recuperados do binding com o objeto possui propriedades especias do Firebase DataSnapshot.

| Proriedade |                    | 
| ---------|--------------------| 
| $key     | A chave de cada registro. É o caminho de registro no nosso banco de dados, seria o mesmo que o retorno do método `ref.key()`.|
| $value   | Se os dados armazenados nesse né filho é um primitivo (number, string ou boolean), ainda sim o registro será um objeto. O valor primitivo serão armazenados em $value e pode ser alterado e salvo como qualquer outro campo.|


## Recuperando o snapshot

AngularFire2 empacota o Firebase DataSnapshot por padrão, mas você pode obter os dados como o snapshot original, especificando a opção `preservar Snapshot`.

```ts
this.item = af.database.object('/item', { preserveSnapshot: true });
this.item.subscribe(snapshot => {
  console.log(snapshot.key)
  console.log(snapshot.val())
});
```

## Consultas (Query)?

Devido ao `FirebaseObjectObservable` sincronizar objetos do banco de dados em tempo real, não é possivel realizar uma classificação por . Por exemplo quando nescessita de filtrar e classificar um objeto, afeta o retorno da consulta, no entando o objeto do retorno é um Json, assim a ordem de retorno não será obedecida. Porem para realizar operações que exige ordenação, provavelmente você precisa de uma lista. (3-recuperando-dados-por-lista.md)

Para filtrar dados veja o topico (4-consultando-listas.md). 


###[Proximo tópico: recuperando-dados-por-lista](3-recuperando-dados-por-lista.md)
