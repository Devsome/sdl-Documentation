title: SRO Database
template: extrahead.html


#### Procedure
For the ranking and other stuff you need to edit your current Silkroad procedures.
After all the `set` variables you need to paste this stuff
`SRO_VT_LOG`.`_AddLogChar`

Modify the procedure on `SRO_VT_LOG`.`_AddLogChar` adding the following like so:
```sql
CREATE   procedure [dbo].[_AddLogChar]
    @CharID		int,
    @EventID		tinyint,
    @Data1		int,
    @Data2		int,
    @strPos		varchar(64),
    @Desc		varchar(128)
as
EXEC SRO_WEB_LARAVEL.dbo._WebProcess @CharID = @CharID,@EventID = @EventID,@DESC = @Desc,@strPos = @strPos
```

Important is to add this `EXEC SRO_WEB_LARAVEL.dbo._WebProcess @CharID = @CharID,@EventID = @EventID,@DESC = @Desc,@strPos = @strPos` right after the `as`

Now modify the `_AddLogSiegeFortress` procedure and add the following at the end:

```sql
DECLARE @FWStatus VARCHAR(10) = (CASE WHEN @EventID = 1 THEN 'Running' WHEN @EventID = 2 THEN 'Stopped' ELSE 'Unknown' END)
IF (@EventID IN(1,2))
BEGIN
IF NOT EXISTS(SELECT * FROM SRO_WEB_LARAVEL.dbo._FortressStatus WHERE Status LIKE 'Running' OR Status LIKE 'Stopped')
BEGIN
TRUNCATE TABLE SRO_WEB_LARAVEL.dbo._FortressStatus
INSERT INTO SRO_WEB_LARAVEL.dbo._FortressStatus (Status,Date) VALUES (@FWStatus,GETDATE())
END
ELSE
UPDATE SRO_WEB_LARAVEL.dbo._FortressStatus SET Status = @FWStatus, Date = GETDATE()
END
```

#### Adding ItemPoints

Open your Silkroad Database query and run this commands.
```SQL
ALTER TABLE dbo._Char ADD ItemPoints int NOT NULL DEFAULT 0;
ALTER TABLE dbo._Guild ADD ItemPoints int NOT NULL DEFAULT 0;
```

That stuff allows the Database to fill the Itempoints to the ranking.
