# Artisan
## Introduction

Artisan is the command-line interface. Artisan exists at the root of your application as the artisan script and provides several helpful commands that can assist you while you build your application. To view a list of all available Artisan commands, you may use the list command:

```sh
./artisan
```

Every command also includes a "help" screen which displays and describes the command's available arguments and options. To view a help screen, add --help option to the command:

```sh
./artisan make:controller --help
```

## Writing Commands
In addition to the commands provided with Artisan, you may build your custom commands. Commands are stored in the app/Console/Commands directory.

## Generating Commands
To create a new command, you may use the make:command Artisan command. This command will create a new command class in the app/Console/Commands directory.

> Please note that you `must` specify `Command` postfix when you are creating commands, otherwise the command will `not` be registered.
```sh
./artisan make:command DoSomeExampleExportCommand
```

### Command Structure
After generating your command, you should define appropriate values for the signature and description properties of the class. These properties will be used when displaying your command on the list screen. The handle method will be called when your command is executed. You may place your command logic in this method.

Let's take a look at an example command.

```typescript
import commander from 'commander';
import BaseCommand, { OptionType } from 'sosise-core/build/Command/BaseCommand';

export default class DoSomeExampleExportCommand extends BaseCommand {
    /**
     * Command name
     */
    protected signature: string = 'example:command';

    /**
     * Command description
     */
    protected description: string = 'This is an example command';

    /**
     * When command is executed prevent from double execution
     */
    protected singleExecution: boolean = false;

    /**
     * Command options
     */
    protected options: OptionType[] = [
        // Options can be boolean
        { flag: '-d, --debug', description: 'Show debug information', required: false },

        // Options can pass some values
        { flag: '-s, --since <since>', description: 'Report since', default: '29.12.2020', required: false },
    ];

    /**
     * Execute the console command
     */
    public async handle(cli: commander.Command): Promise<void> {
        // When debug option is true
        if (cli.debug) {
            console.log('Debug option provided');
        }

        console.log('Example export since ' + cli.since);
    }
}
```

> For greater code reuse, it is good practice to keep your console commands light and let them defer to application services to accomplish their tasks.

## Defining Input Expectations
### Options
Options, like arguments, are another form of user input. Options are prefixed by two hyphens (--) when they are provided via the command line. There are two types of options: those that receive a value and those that don't. Options that don't receive a value serve as a boolean "switch". Let's take a look at an example of this type of option:

```typescript
/**
 * Command options
 */
protected options: OptionType[] = [
    // Options can be boolean
    { flag: '-d, --debug', description: 'Show debug information' },
];

/**
 * Execute the console command
 */
public async handle(cli: commander.Command): Promise<void> {
    console.log(cli.debug);
}
```

In this example, the --debug or -d switch may be specified when calling the Artisan command. If the --debug switch is passed, the value of the option will be `true`. Otherwise, the value will be `undefined`:

```sh
./artisan example:command --debug
```

### Options With Values
Next, let's take a look at an option that expects a value. Example below:

```typescript
/**
 * Command options
 */
protected options: OptionType[] = [
    // Options can pass some values
    { flag: '-s, --since <since>', description: 'Report since' },
];

/**
 * Execute the console command
 */
public async handle(cli: commander.Command): Promise<void> {
    console.log(cli.since);
}
```

In this example, the user may pass a value for the option like so. If the option is not specified when invoking the command, its value will be `undefined`.

```sh
./artisan example:command --since 01.01.2021
```

### Options with default values
Next, let's take a look at an options that have default value. Example below:

```typescript
/**
 * Command options
 */
protected options: OptionType[] = [
    // Options can be boolean
    { flag: '-d, --debug', description: 'Show debug information', default: false },

    // Options can pass some values
    { flag: '-s, --since <since>', description: 'Report since', default: '29.12.2020' },

    // Options can pass some values
    { flag: '-t, --to <to>', description: 'Report to', default: dayjs().format('YYYY-MM-DD') },
];

/**
 * Execute the console command
 */
public async handle(cli: commander.Command): Promise<void> {
    console.log(cli.debug, cli.since, cli.to);
}
```

In this example, if you don't pass options to the command default values will be taken `false`, `29.12.2020`, `%today-date%`

### Required options
Next, let's take a look at an `required` option. Example below:

```typescript
/**
 * Command options
 */
protected options: OptionType[] = [
    // Options can pass some values
    { flag: '-t, --to <to>', description: 'Report to', required: true },
];

/**
 * Execute the console command
 */
public async handle(cli: commander.Command): Promise<void> {
    console.log(cli.to);
}
```

> Please note that an option is only then required when it's `required` property isset to true and it does not has default value

## Preventing double execution of your commands
```typescript
/**
 * When command is executed prevent from double execution
 */
protected singleExecution: boolean = true;
```

In this example, you would not be able to run the command while the last command is running, just like that!
