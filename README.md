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

**Root Enviroment Injector (classic), Element Injector, Module Injector, Platform Enviroment Injector (error)**

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

***Important thing, that if using Element Injector, the service is availabe only in the that component and its child components***

***Every instance of the component will get its own service instance***

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

***YOU CANNOT USE ELEMENT INJECTION FOR INJECTING SERVICE INTO SERVICE, BECAUSE ONLY COMPONENTS AND DIRECTIVE DO REACH OUT TO ELEMENT INJECTOR***