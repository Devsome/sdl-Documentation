title: Unique
template: extrahead.html

To make it work follow these steps.

1. Get the newest file changes by `git pull`
2. In the `config` folder you will find a `unique.example.php` file. Rename it to `unique.php`.
3. Run `php artisan migrate` to get the newest Unique Table + Procedure.
    1. Table `SRO_VT_LOG.uniquekilllogs` should be created.
    2. Procedure `SRO_VT_LOG._ExecUniquekilllogs` should be created.
4. Run `composer dump-autoload`

You are probably wondering how I enter the Unique Kills now. For this you have to execute this Exec command.
It does not matter if you send the MobName or real name.

Here are two examples:

```sql
EXEC _ExecUniquekilllogs @CharName16 = 'Devsome', @UniqueName = 'MOB_KK_ISYUTARU';
```

```sql
EXEC _ExecUniquekilllogs @CharName16 = 'Devsome', @UniqueName = 'Captain Ivy';
```

As long as the MobName or real name is stored in the Config file, they will be found.


> Just to remind you, you need any second tool to execute the procedure.
