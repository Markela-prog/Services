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
