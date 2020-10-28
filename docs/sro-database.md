title: SRO Database
template: extrahead.html


#### Procedure
For the ranking and other stuff you need to edit your currenct Silkroad procedures.
After all the `set` variables you need to paste this stuff
`SRO_VT_LOG`.`_AddLogChar`

```sql
IF(@EventID = 4 or @EventID = 6)
BEGIN
  SET NOCOUNT ON;
    INSERT INTO loginhistory ([CharID], [status])
    VALUES (@CharID, @EventId)
  IF EXISTS (SELECT 1 FROM onlineofflinelog WHERE CharID = @CharID)
    UPDATE onlineofflinelog
    SET    status = @EventID
    WHERE	CharID = @CharID
  ELSE
    INSERT INTO onlineofflinelog ([CharID],  [status])
    VALUES      (@CharID, @EventID)
END
IF (@EventID = 6)
BEGIN
    UPDATE [SRO_VT_SHARD].[dbo]._Char
        set ItemPoints = (  
            SELECT
            SUM(
               CASE
                   WHEN Common.CodeName128 LIKE '%_A_RARE' THEN ReqLevel1 + 5
                   WHEN Common.CodeName128 LIKE '%_B_RARE' THEN ReqLevel1 + 10
                   WHEN Common.CodeName128 LIKE '%_C_RARE' THEN ReqLevel1 + 15
               ELSE ReqLevel1
               END
       ) + SUM(ISNULL(Binding.nOptValue, 0)) + SUM(ISNULL(OptLevel, 0)) AS ItemPoints       

        FROM [SRO_VT_SHARD].[dbo].[_Inventory] as inventory
        join [SRO_VT_SHARD].[dbo]._Items as Items on Items.ID64  = inventory.ItemID
        join [SRO_VT_SHARD].[dbo]._RefObjCommon as Common on Items.RefItemId  = Common.ID
        left join [SRO_VT_SHARD].[dbo]._BindingOptionWithItem as Binding on Binding.nItemDBID = Items.ID64
        where
            inventory.slot < 13 and
            inventory.slot != 8 and
            inventory.slot != 7 and
            inventory.CharID = _Char.CharID
    ) WHERE _Char.CharID = @CharID

    Declare @GuildID int;
    SELECT @GuildID = GuildID FROM [SRO_VT_SHARD].[dbo]._Char WHERE _Char.CharID = @CharID

    IF (@GuildID > 0)
    BEGIN
        UPDATE [SRO_VT_SHARD].[dbo]._Guild
          set ItemPoints = (
          SELECT
            SUM(Char.ItemPoints) as ItemPoints
            FROM [SRO_VT_SHARD].[dbo]._Char as Char
            where
                Char.GuildID = _Guild.ID
        ) WHERE _Guild.ID = @GuildID
    END
END
```

Also you need to add this to the `_AddLogChar` procedure for the Job and PvP stats

```sql
--Character kills table recorder (finish) , all credits goes to #HB
IF (@EventID = 20) -- PVP
BEGIN
    IF ( @desc LIKE '%Trader, Neutral, no freebattle team%'    -- Trader
        OR @desc LIKE '%Hunter, Neutral, no freebattle team%'    -- Hunter
        OR @desc LIKE '%Robber, Neutral, no freebattle team%'    -- Thief
        OR @desc like '%no job, Neutral, %no job, Neutral%'    -- Free PVP
    )
    BEGIN
        -- Get killer name
        DECLARE @killername VARCHAR(512) = @desc
        DECLARE @killeriD INT = 0
		SELECT  @killername = REPLACE(@killername, LEFT(@killername, CHARINDEX('(', @killername)),'')
		SELECT  @killername = REPLACE(@killername, RIGHT(@killername, CHARINDEX(')', REVERSE(@killername))),'')
        SELECT @killeriD = CharID FROM [SRO_VT_SHARD].[dbo].[_Char] WHERE CharName16 = @killername
        -- Get job type
        DECLARE @jobString VARCHAR(10) = LTRIM(RTRIM(SUBSTRING (@desc, 5, 7)))
        DECLARE @jobType INT = CASE
            WHEN @jobString LIKE 'Trader' THEN 1
            WHEN @jobString LIKE 'Robber' THEN 2
            WHEN @jobString LIKE 'Hunter' THEN 3
            ELSE 0 END
        -- Delete original log
        DELETE FROM _LogEventChar WHERE CharID = @CharID AND EventID = 20
            AND (strDesc LIKE '%Trader, Neutral, no freebattle team%'
            OR strDesc LIKE '%Hunter, Neutral, no freebattle team%'
            OR strDesc LIKE '%Robber, Neutral, no freebattle team%'
            OR @desc like '%no job, Neutral, %no job, Neutral%')
        -- Get additional info for notice message
        DECLARE @jobDesc VARCHAR(32) = CASE WHEN @jobType BETWEEN 1 AND 3 THEN 'Job Conflict' ELSE 'Free PVP' END
        DECLARE @strDesc VARCHAR(512)
        IF  (@jobString LIKE 'Trader' OR @jobString LIKE 'Robber' OR @jobString LIKE 'Hunter')
        BEGIN
            -- If it's a Job Kill, then write character nicknames
            DECLARE @killerNickName VARCHAR(64) = (SELECT NickName16 FROM [SRO_VT_SHARD].[dbo].[_Char] WHERE CharID = @killeriD)
            DECLARE @CharnickName VARCHAR(64) = (SELECT NickName16 FROM [SRO_VT_SHARD].[dbo].[_Char] WHERE CharID = @CharID)
            SET @strDesc = '[' + @killerNickName + '] has just killed [' + @CharnickName + '] in [' + @jobDesc + '] mode on [' + CONVERT(NVARCHAR(30), GETDATE(), 0) + ']'
        END

        ELSE BEGIN
            -- If it's normal PVP Kill, write real character names
            SET @strDesc = '[' + @killername + '] has just killed [' + @CharName + '] in [' + @jobDesc + '] mode on [' + CONVERT(NVARCHAR(30), GETDATE(), 0) + ']'
        END
        -- Update the log
        INSERT INTO pvp_records VALUES (@killername, @killeriD, @CharName, @CharID, @jobType, @strPos, @strDesc, GETDATE())
    END
END
```

That inserts a record into the `SRO_VT_LOG`.`onlineofflinelog` & `SRO_VT_LOG`.`loginhistory`.<br>
If you think, that you are missing that table. Do the next step and the Table exist. <br>
Also the EventId = 6 part is for the ranking of your players.

#### Adding ItemPoints

Open your Silkroad Database query and run this commands.
```SQL
ALTER TABLE dbo._Char ADD ItemPoints int NOT NULL DEFAULT 0;
ALTER TABLE dbo._Guild ADD ItemPoints int NOT NULL DEFAULT 0;
```

That stuff allows the Database to fill the Itempoints to the ranking.
