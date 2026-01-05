## Código Original

**user.service.ts**

```typescript
import { Injectable } from '@angular/core';
import { AngularFirestore } from '@angular/fire/firestore';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  constructor(private firestore: AngularFirestore) {}

  getUsers() {
    return this.firestore.collection('users').valueChanges();
  }

  async getUserById(id: string) {
    const snapshot = await this.firestore.collection('users').doc(id).get().toPromise();
    return snapshot?.data();
  }

  updateUser(id: string, data: any) {
    return this.firestore.collection('users').doc(id).set(data);
  }

  createUser(user: any) {
    return this.firestore.collection('users').add(user);
  }
}
```

**user-list.component.ts**

```typescript
import { Component, OnInit } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-user-list',
  template: `
    <div *ngFor="let user of users">
      {{ user.name }} - {{ user.email }}
    </div>
  `
})
export class UserListComponent implements OnInit {
  users: any[] = [];

  constructor(private userService: UserService) {}

  ngOnInit() {
    this.userService.getUsers().subscribe(users => {
      this.users = users;
    });
  }
}
```

---

## O que encontrei

Olhando o código, alguns pontos me chamaram atenção:

**Problemas sérios:**
- O `updateUser` usa `set()` ao invés de `update()` - isso vai sobrescrever o documento inteiro, apagando campos que não foram passados
- Memory leak no component - a subscription do Firestore nunca é cancelada quando o componente é destruído
- `any` em todo lugar - perde todo o benefício do TypeScript

**Coisas que poderiam melhorar:**
- `valueChanges()` não retorna o ID do documento, o que complica na hora de fazer update/delete
- Nenhum tratamento de erro em lugar nenhum
- Sem feedback de loading pro usuário
- O `*ngFor` não tem `trackBy`, o que pode impactar performance em listas maiores

---

## Código Refatorado

### user.model.ts

Criei uma interface pra tipar os dados. O `any` funciona, mas a gente perde autocomplete, validação em tempo de compilação, e fica mais difícil de entender o que o código espera receber.

```typescript
export interface User {
  id?: string;
  name: string;
  email: string;
  createdAt?: Date;
  updatedAt?: Date;
}

// Pra criação, não faz sentido passar id ou timestamps - o sistema gera
export type CreateUserDTO = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;

// Pra update, todos os campos são opcionais (atualização parcial)
export type UpdateUserDTO = Partial<Omit<User, 'id'>>;
```

---

### user.service.ts

```typescript
import { Injectable } from '@angular/core';
import { AngularFirestore, DocumentReference } from '@angular/fire/firestore';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';
import { User, CreateUserDTO, UpdateUserDTO } from './user.model';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  // Evita string 'users' espalhada pelo código
  private readonly COLLECTION_NAME = 'users';

  constructor(private firestore: AngularFirestore) {}

  /**
   * Busca todos os usuários.
   * 
   * Mudanças:
   * - Adicionei tipagem no retorno
   * - O { idField: 'id' } faz o valueChanges incluir o ID do documento,
   *   que é necessário pra fazer update/delete depois
   * - catchError pra não quebrar silenciosamente
   */
  getUsers(): Observable<User[]> {
    return this.firestore
      .collection<User>(this.COLLECTION_NAME)
      .valueChanges({ idField: 'id' })
      .pipe(
        catchError(error => {
          console.error('[UserService] Erro ao buscar usuários:', error);
          return throwError(() => new Error('Falha ao carregar usuários'));
        })
      );
  }

  /**
   * Busca usuário pelo ID.
   * 
   * Retorna null se não existir (antes retornava undefined sem indicação clara).
   * Inclui o ID no objeto retornado pra manter consistência com getUsers().
   */
  async getUserById(id: string): Promise<User | null> {
    try {
      const snapshot = await this.firestore
        .collection<User>(this.COLLECTION_NAME)
        .doc(id)
        .get()
        .toPromise();

      if (!snapshot?.exists) {
        return null;
      }

      return { id: snapshot.id, ...snapshot.data() } as User;
    } catch (error) {
      console.error(`[UserService] Erro ao buscar usuário ${id}:`, error);
      throw new Error('Falha ao buscar usuário');
    }
  }

  /**
   * Atualiza usuário.
   * 
   * IMPORTANTE: Troquei set() por update().
   * 
   * O set() sobrescreve o documento inteiro. Se você tem um usuário com
   * { name, email, role } e chama updateUser(id, { name: 'Novo Nome' }),
   * com set() você perde email e role. Com update() só o name é alterado.
   * 
   * Também adicionei updatedAt automático pra rastreabilidade.
   */
  async updateUser(id: string, data: UpdateUserDTO): Promise<void> {
    try {
      await this.firestore
        .collection(this.COLLECTION_NAME)
        .doc(id)
        .update({
          ...data,
          updatedAt: new Date()
        });
    } catch (error) {
      console.error(`[UserService] Erro ao atualizar usuário ${id}:`, error);
      throw new Error('Falha ao atualizar usuário');
    }
  }

  /**
   * Cria usuário.
   * 
   * Mudanças:
   * - Retorna só o ID (string) ao invés do DocumentReference do Firebase
   * - Adiciona timestamps de criação automaticamente
   */
  async createUser(user: CreateUserDTO): Promise<string> {
    try {
      const docRef: DocumentReference = await this.firestore
        .collection(this.COLLECTION_NAME)
        .add({
          ...user,
          createdAt: new Date(),
          updatedAt: new Date()
        });
      
      return docRef.id;
    } catch (error) {
      console.error('[UserService] Erro ao criar usuário:', error);
      throw new Error('Falha ao criar usuário');
    }
  }

  /**
   * Adicionei delete que não existia no original.
   */
  async deleteUser(id: string): Promise<void> {
    try {
      await this.firestore
        .collection(this.COLLECTION_NAME)
        .doc(id)
        .delete();
    } catch (error) {
      console.error(`[UserService] Erro ao deletar usuário ${id}:`, error);
      throw new Error('Falha ao deletar usuário');
    }
  }
}
```

---

### user-list.component.ts

```typescript
import { Component, OnInit, OnDestroy, ChangeDetectionStrategy } from '@angular/core';
import { Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';
import { UserService } from './user.service';
import { User } from './user.model';

@Component({
  selector: 'app-user-list',
  // OnPush melhora performance - só re-renderiza quando inputs mudam ou eventos disparam
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div *ngIf="isLoading" class="loading">
      Carregando usuários...
    </div>

    <div *ngIf="error" class="error">
      {{ error }}
      <button (click)="loadUsers()">Tentar novamente</button>
    </div>

    <div *ngIf="!isLoading && !error && users.length === 0" class="empty-state">
      Nenhum usuário encontrado.
    </div>

    <!-- trackBy evita recriar todos os elementos quando a lista muda -->
    <div *ngFor="let user of users; trackBy: trackByUserId" class="user-item">
      {{ user?.name }} - {{ user?.email }}
    </div>
  `
})
export class UserListComponent implements OnInit, OnDestroy {
  users: User[] = [];
  isLoading = false;
  error: string | null = null;

  /**
   * Esse Subject é usado pra cancelar subscriptions quando o componente morre.
   * 
   * O problema do código original: o Observable do Firestore é "hot", ele continua
   * emitindo valores mesmo depois que o componente foi destruído. Sem unsubscribe:
   * - Memória não é liberada (memory leak)
   * - Callbacks rodam em componente que não existe mais
   * - Pode dar erro "Cannot read property of undefined"
   */
  private destroy$ = new Subject<void>();

  constructor(private userService: UserService) {}

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.isLoading = true;
    this.error = null;

    this.userService.getUsers()
      .pipe(
        // Quando destroy$ emitir (no ngOnDestroy), essa subscription é cancelada
        takeUntil(this.destroy$)
      )
      .subscribe({
        next: (users) => {
          this.users = users;
          this.isLoading = false;
        },
        error: (err) => {
          this.error = err.message || 'Erro ao carregar usuários';
          this.isLoading = false;
          console.error('[UserListComponent] Erro:', err);
        }
      });
  }

  trackByUserId(index: number, user: User): string {
    return user.id || index.toString();
  }

  /**
   * Cleanup das subscriptions. Sem isso = memory leak.
   */
  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

---

## Resumo

| Problema | Solução |
|----------|---------|
| Memory leak (subscription não cancelada) | `takeUntil` + `Subject` no `ngOnDestroy` |
| `set()` apagando campos no update | Trocado por `update()` |
| `any` em tudo | Interfaces tipadas |
| `valueChanges()` sem ID | `{ idField: 'id' }` |
| Sem tratamento de erro | `catchError`, try/catch, estados de erro no component |
| Sem loading/feedback | Estados `isLoading` e `error` |
| `*ngFor` sem trackBy | `trackByUserId` |
| CRUD incompleto | Adicionado `deleteUser()` |

---

## Alternativa mais limpa com async pipe

Se quiser simplificar ainda mais, dá pra usar o `async` pipe direto no template. O Angular gerencia a subscription sozinho:

```typescript
@Component({
  selector: 'app-user-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <ng-container *ngIf="users$ | async as users; else loading">
      <div *ngIf="users.length === 0">Nenhum usuário encontrado.</div>
      <div *ngFor="let user of users; trackBy: trackByUserId">
        {{ user.name }} - {{ user.email }}
      </div>
    </ng-container>
    <ng-template #loading>Carregando...</ng-template>
  `
})
export class UserListComponent {
  users$ = this.userService.getUsers();
  
  constructor(private userService: UserService) {}
  
  trackByUserId(index: number, user: User): string {
    return user.id || index.toString();
  }
}
```

Assim não precisa nem do `OnDestroy`.
