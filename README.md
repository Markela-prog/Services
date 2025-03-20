# ServicesDeepDive

The important thing, is that we want to use same service instance for multiple components, so we cannot create a separate instances in each component, instead we use Dependency Injection

## Dependency Injection

You do not create service instances yourself - instead, you request them from Angular

<br>

```
constructor(tService: TasksService) {

  }
```

TasksService - the type that serves as the so-called "Injection Token" which is used by Angular to identify the "thing" (e.g., the service) it should create and inject

<br>

One of the possible way (longer):

```
export class NewTaskComponent {
  private formEl = viewChild<ElementRef<HTMLFormElement>>('form');
  private tasksService: TasksService;

  constructor(tService: TasksService) {
    this.tasksService = tService;
  }

  onAddTask(title: string, description: string) {
    this.tasksService.addTask({title, description})
    this.formEl()?.nativeElement.reset();
  }
}
```

We could simplify the code (The Angular creates property by itself):

```
export class NewTaskComponent {
  private formEl = viewChild<ElementRef<HTMLFormElement>>('form');

  constructor(private tasksService: TasksService) {

  }

  onAddTask(title: string, description: string) {
    this.tasksService.addTask({title, description})
    this.formEl()?.nativeElement.reset();
  }
}
```

Alternative way is using inject() function:

```
export class TasksListComponent {
  private tasksService = inject(TasksService);
  selectedFilter = signal<string>('all');
  tasks = this.tasksService.allTasks;

  onChangeTasksFilter(filter: string) {
    this.selectedFilter.set(filter);
  }
}
```

<br>

## Injections

Angular has injectors to register values that can be requested by components, directives, services.

**null injector above the platform (error generating if no values)**

**Root Enviroment Injector (classic), Element Injector(The first one to be called), Module Injector, Platform Enviroment Injector (Provide value for multiple apps in 1 Angular project)**

We can provide services without @Injectable, but with providers in main.ts:

```
bootstrapApplication(AppComponent, {
    providers: [TasksService]
}).catch((err) => console.error(err));
```

The difference is that config object does not allow for tree shaking of the service. Which means that Angular tries to optimize your code as much as possible once you prepared for deployment and during optimization process it tries to throw away any code thats not being use or not needed initially when app starts.

<hr>

We can provide services via Element Injector:

```
@Component({
  selector: 'app-tasks',
  standalone: true,
  templateUrl: './tasks.component.html',
  imports: [NewTaskComponent, TasksListComponent],
  providers: [TasksService]
})
```

**_Important thing, that if using Element Injector, the service is availabe only in the that component and its child components_**

**_Every instance of the component will get its own service instance_**

<hr>

You can inject services into service:

```
@Injectable({
  providedIn: 'root',
})
export class LoggingService {
  log(message: string) {
    const timeStamp = new Date().toLocaleTimeString();
    console.log(`[${timeStamp}]: ${message}`);
  }
}
```

and use it in another service

```
export class TasksService {
  private tasks = signal<Task[]>([]);
  private loggingService = inject(LoggingService);

  allTasks = this.tasks.asReadonly();

  addTask(taskData: { title: string; description: string }) {
    const newTask: Task = {
      ...taskData,
      id: Math.random().toString(),
      status: 'OPEN',
    };
    this.tasks.update((oldTasks) => [...oldTasks, newTask]);
    this.loggingService.log('ADDED TASK WITH TITLE ' + taskData.title);
  }

  updateTasksService(taskId: string, newStatus: TaskStatus) {
    this.tasks.update((oldTasks) =>
      oldTasks.map((task) =>
        task.id === taskId ? { ...task, status: newStatus } : task
      )
    );
    this.loggingService.log('CHANGED TASK STATUS TO ' + newStatus);
  }
}
```

**_YOU CANNOT USE ELEMENT INJECTION FOR INJECTING SERVICE INTO SERVICE, BECAUSE ONLY COMPONENTS AND DIRECTIVE DO REACH OUT TO ELEMENT INJECTOR_**

## Custom DI token

By default the Service class name is the DI token that we pass to inject function

But we can create our own DI token:

```
import { InjectionToken } from '@angular/core';
import { TasksService } from './app/tasks/tasks.service';

const TasksServiceToken = new InjectionToken<TasksService>('tasks-service-token');

bootstrapApplication(AppComponent, {
    providers: [{provide: TasksServiceToken, useClass: TasksService}]
}).catch((err) => console.error(err));
```

## Non-class value injection

You can not only inject classes but also inject values

```
import { InjectionToken, Provider } from "@angular/core";

export type TaskStatus = 'OPEN' | 'IN_PROGRESS' | 'DONE';

type TaskStatusOptions = {
  value: 'open' | 'in-progress' | 'done';
  taskStatus: TaskStatus;
  text: string
}[];


export const TASK_STATUS_OPTIONS = new InjectionToken<TaskStatusOptions>('task-status-options');

export const TaskStatusOptions: {
  value: 'open' | 'in-progress' | 'done';
  taskStatus: TaskStatus;
  text: string
}[] = [
  {
    value: 'open',
    taskStatus: 'OPEN',
    text: 'Open'
  },
  {
    value: 'in-progress',
    taskStatus: 'IN_PROGRESS',
    text: 'In-progress'
  },
  {
    value: 'done',
    taskStatus: 'DONE',
    text: 'Done'
  },
];

export const taskStatusOptionsProvider: Provider = {
  provide: TASK_STATUS_OPTIONS,
  useValue: TaskStatusOptions
}

export interface Task {
  id: string;
  title: string;
  description: string;
  status: TaskStatus;
}
```

then in component class

```
@Component({
  selector: 'app-tasks-list',
  standalone: true,
  templateUrl: './tasks-list.component.html',
  styleUrl: './tasks-list.component.css',
  imports: [TaskItemComponent],
  providers: [taskStatusOptionsProvider]
})
export class TasksListComponent {
  private tasksService = inject(TasksService);
  private selectedFilter = signal<string>('all');
  taskStatusOptions = inject(TASK_STATUS_OPTIONS);
```

and in template

```
<header>
  <h2>My Tasks</h2>
  <p>
    <select (change)="onChangeTasksFilter(filter.value)" #filter>
      <option value="all">All</option>
      @for (option of taskStatusOptions; track option.value) {
        <option [value]="option.value">{{ option.text }}</option>
      }
    </select>
  </p>
</header>
```

Also we can use it in child component

```
<form (ngSubmit)="onChangeTaskStatus(task().id, status.value)">
    <select #status>
      @for (option of taskStatusOptions; track option.value){
        <option [value]="option.value" [selected]="task().status === option.taskStatus">{{ option.text }}</option>
      }
    </select>
    <p>
      <button>Change Status</button>
    </p>
  </form>
```

