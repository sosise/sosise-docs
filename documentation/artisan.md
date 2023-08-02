# Artisan

## Introduction

Artisan is the command-line interface for Sosise applications. It provides a set of helpful commands that can assist you while building and managing your application. To view a list of all available Artisan commands.

```sh
./artisan
```

Each command also includes a "help" screen that displays and describes the command's available arguments and options. To view the help screen, add the `--help` option to the command:

```sh
./artisan make:controller --help
```

## Writing Custom Commands

In addition to the built-in commands provided by Artisan, you can create your custom commands to perform specific tasks or automate actions. Custom commands are stored in the `app/Console/Commands` directory.

## Generating Custom Commands

To create a new custom command, you can use the `make:command` Artisan command. This command will generate a new command class in the `app/Console/Commands` directory.

> Important: Make sure to specify the `Command` postfix when creating custom commands; otherwise, the command will not be registered properly.

```sh
./artisan make:command DoSomeExampleExportCommand
```

### Command Structure

After generating your command, you should define the appropriate values for the `signature` and `description` properties of the class. These properties will be used when displaying your command on the list screen. The `handle` method will be called when your command is executed, and you can place your command logic in this method.

Let's take a look at an example command:

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
     * When the command is executed, prevent it from double execution
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
        // Check if the debug option is provided
        if (cli.debug) {
            console.log('Debug option provided');
        }

        console.log('Example export since ' + cli.since);
    }
}
```

> For better code reuse, it is a good practice to keep your console commands light and let them defer to application services to accomplish their tasks.

## Defining Input Expectations

### Options

Options, like arguments, are another form of user input. Options are prefixed by two hyphens (`--`) when provided via the command line. There are two types of options: those that receive a value and those that don't. Options that don't receive a value serve as a boolean "switch". Let's take a look at an example of this type of option:

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

In this example, the `--debug` or `-d` switch may be specified when calling the Artisan command. If the `--debug` switch is passed, the value of the option will be `true`. Otherwise, the value will be `undefined`:

```sh
./artisan example:command --debug
```

### Options with Values

Next, let's take a look at an option that expects a value:

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

In this example, the user may pass a value for the option like this. If the option is not specified when invoking the command, its value will be `undefined`.

```sh
./artisan example:command --since 01.01.2021
```

### Options with Default Values

Next, let's take a look at options that have default values:

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

In this example, if you don't pass options to the command, default values will be taken: `false`, `'29.12.2020'`, and today's date.

### Required Options

Next, let's take a look at a required option:

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

> Please note that an option is only required when its `required` property is set to `true`, and it does not have a default value.

## Preventing Double Execution of Your Commands

To prevent a command from being executed more than once simultaneously, you can use the `singleExecution` property:

```typescript
/**
 * When the command is executed, prevent it from double execution
 */
protected singleExecution: boolean = true;
```

In this example, you would not be able to run the command while the last command is still running.