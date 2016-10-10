# 1. Instalação e Configuração

> Com Angular-Cli fica a mais facil começar a usar o AngularFire2(https://github.com/angular/angular-cli).Para começar basta seguir estes 10 passos abaixo. Fique tranquilo estamos fazendo o possivel para torna-lo mais curto.

### 0. Pré-Requisitos

Antes de começar a instalação do AngularFire2, certifique-se de que possui instalado a ultima versão do Angular-Cli.
Verifique a versão instalada utilizando o comando 'ng -V', verifique a versão é superior á angular-cli: `1.x.x-beta.14`.

Caso contrario você preciza fazer o seguinte: 

```bash
# em caso de versão incompatível
npm uninstall -g angular-cli

# reinstalar versão limpa
npm install -g angular-cli 
```
Você precisa do Angular-Cli, typings e TypeScript.

```bash
npm install -g angular-cli  
# ou instale localmente
npm install angular-cli --save-dev
# certifique de instalar typings
npm install -g typings 
npm install -g typescript
```

### 1. Criando o Projeto

```bash
ng new <project-name>
cd <project-name>
```
O Angular-Cli irá configurar um estrutura de projeto na versão mais recente do angular.

### 2. Teste e Configuração

```bash
ng serve
abrir http://localhost:4200
```
Você deverá ver uma mensagem *App works!*

### 3. Instalando AngularFire 2 e Firebase

```bash
npm install angularfire2 firebase --save
```

Agora que você já possui um projeto configurado, instale o AngularFire 2 e Firebase atravez do NPM.

### 4. Configurando @NgModule


Abra o arquivo `/ src / app / app.module.ts`, importe os providers do Firebase e especifique qual sua configuração do Firebase.
Podemos encontrar essas informações no [console do Firebase] na Web (https://console.firebase.google.com)


```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { AngularFireModule } from 'angularfire2';

// Deve exportar a configuração
export const firebaseConfig = {
  apiKey: '<your-key>',
  authDomain: '<your-project-authdomain>',
  databaseURL: '<your-database-URL>',
  storageBucket: '<your-storage-bucket>'
};

@NgModule({
  imports: [
    BrowserModule,
    AngularFireModule.initializeApp(firebaseConfig)
  ],
  declarations: [ AppComponent ],
  bootstrap: [ AppComponent ]
})
export class AppModule {}

```

### 5. Injete AngularFire

Abra o arquivo `/src/app/app.component.ts`, para tudo funcionar corretamente modifique/exclua os arquivos de testes (Testes são importantes, você sabe disso.)


```ts
import { Component } from '@angular/core';
import { AngularFire, FirebaseListObservable } from 'angularfire2';

@Component({
 
  selector: 'app-root',
  templateUrl: 'app.component.html',
  styleUrls: ['app.component.css']
})
export class <MyApp>Component {
  constructor(af: AngularFire) {
    
  }
}

```

### 6. Bind da lista

No arquivo `/src/app/app.component.ts`:

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

Abra o arquivo `/src/app/app.component.html`:

```html
<ul>
  <li class="text" *ngFor="let item of items | async">
    {{item.$value}}
  </li>
</ul>
```

### 7. Rodando seu app

```bash
ng serve
```
Execute o comando para iniciar o servidor e navegue até `localhost: 4200` em seu navegador.

E é isso! Qualquer problema que encontrar registre uma issue para nos avisar.

###[Próximo passo: recuperação de dados como objeto](2-recuperacao-de-dados-como-objeto.md)

### Solução de problemas

#### 1. Não é possível encontrar o namespace 'firebase'.

Menssagem no console: Cannot find namespace 'firebase'.

Se você tiver esse erro ao rodar o comando `ng serve`, abra o arquivo `src/tsconfig.json` e adicione  "firebase" no array "types", observe o exemplo abaixo:

```json
{
  "compilerOptions": {
    ...
    "typeRoots": [
      "../node_modules/@types"
    ],
    
    // ADICIONE ISTO
    "types": [
      "firebase"
    ]
  }
}
```
#### 2. Não é possível encontrar o nome de 'require' (Isto é apenas uma solução temporária para o Angular CLI).

Menssagem no console: Cannot find name 'require'.

If you run into this error while trying to invoke `ng serve`, open `src/typings.d.ts` and add the following two entries as follows:

Se você tiver esse erro ao rodar o comando `ng serve`, abra o arquivo `src/typings.d.ts` e adicione as seguintes declarações:

```bash
declare var require: any;
declare var module: any;
```
