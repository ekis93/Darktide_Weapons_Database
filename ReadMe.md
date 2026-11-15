# Darktide Weapon Database

An SQL Server database containing weapon stats for the game Warhammer 40K: Darktide.

Notice: This is my first database so you may come across things that are not 100% correct in terms of format. Feel free to open an issue if anything seems off.

> **Game version:** 1.13.4<br>
> **Last updated:** 2026-11-15

---

## Contents

| Folder | What it contains | Use if… |
|---|---|---|
| `csv/` | One CSV file per table | You want to open the data in Excel, Python, etc. |
| `sql/` | Table structure and all data | You want to rebuild the tables |

---

## Tables

| Table | Description |
|---|---|
| `Action` | Action names (Strikedown, Vanguard etc), action types (Primary, Secondary etc), Attack chains (first hit, second hit etc) |
| `Class` | Class names |
| `ClassWeapon` | Available weapons for each class |
| `Damage` | Damage breakdown for each weapon. Damage output is determined by the Action, Enemy, Hit and Weapon type  |
| `Enemy` | Enemy types (Unarmoured, Flak Armoured etc) |
| `Hit` | Hit types (Body, Critical etc) |
| `Weapon` | Full weapon names |

---

## Installation

### Option 1: Create tables from file - AttackData_full.sql

**SSMS:** 
1. Connect to the server in Object Explorer.
2. Create database: Right-click Databases -> New Database -> name it -> click OK
3. Open SQL file: File -> Open -> File -> pick your sql file
4. Select database: Toolbar -> database dropdown -> pick your database
5. Execute: Make sure nothing is highlighted -> click Execute (or press F5)

**Other SQL Server tools (DataGrip, Azure Data Studio, sqlcmd):**
- Create an empty database -> use database -> run `AttackData_full.sql`
- In DataGrip: right-click the database -> SQL Scripts -> Run SQL Script
- sqlcmd: `sqlcmd -S <server> -d <database> -i sql/AttackData_full.sql`

> The script is written for Microsoft SQL Server. It will not run on other database systems.


### Option 2: CSV files

The csv/ folder has one file per table. Open them in any spreadsheet program, or load them into the database of your choice. Each file has a header row with the column names.

---

## Example queries


**All attacks for a weapon vs. an enemy type, sorted by max damage (highest first)**
> Chained attacks can share the same damage output (e.g. Attack_chain 1 and 2 may share the same damage values).
```sql
SELECT WeaponName,
       ActionType,
       ActionComboID AS Attack_chain,
       ActionName,
       HitType,
       EnemyType,
       DamageNumbHi
FROM AttackData.Damage
WHERE WeaponName = N'Cadia Mk XIIIg Assault Chainsword'
  AND EnemyType  = N'Carapace Armoured'
ORDER BY DamageNumbHi DESC, ActionType, Attack_chain;
```

**Strongest attack for each given weapon vs. a given enemy type**
> Returns the single highest-damage attack for each listed weapon. If several attacks tie for the top value, the tie-breakers in `ORDER BY` decide which row is shown.

> Can be used for comparing damage output between different marks, for instance:
```sql
SELECT WeaponName,
       ActionType,
       Attack_chain,
       ActionName,
       HitType,
       EnemyType,
       DamageNumbHi
FROM (
         SELECT WeaponName,
                ActionType,
                ActionComboID AS Attack_chain,
                ActionName,
                HitType,
                EnemyType,
                DamageNumbHi,
                ROW_NUMBER() OVER (
                    PARTITION BY WeaponName
                    ORDER BY DamageNumbHi DESC, ActionType, ActionComboID
                    ) AS rn
         FROM AttackData.Damage
         WHERE WeaponName IN (N'Accatran Mk VIc Recon Lasgun',
                              N'Accatran Mk XII Recon Lasgun',
                              N'Accatran Mk XIV Recon Lasgun')
           AND EnemyType  = N'Unarmoured'
     ) AS ranked
WHERE rn = 1
ORDER BY DamageNumbHi DESC, WeaponName;
```

---

## Data source

The damage values were calculated from the game's decompiled scripts in [Aussiemon/Darktide-Source-Code](https://github.com/Aussiemon/Darktide-Source-Code). Weapon names and class availability come from [Games Lantern](https://darktide.gameslantern.com/).

**AI NOTICE:**

The database design (tables, keys and relationships) and the schema script were created by hand. Claude.ai was used to generate the data: it calculated the damage values by running the game’s own weapon stat code from the decompiled Lua files, matched the game’s internal weapon IDs to their in-game names, and generated INSERT scripts from the custom schema script.

## Contributing

Found a wrong value or missing weapon? [Open an issue](../../issues) or submit a pull request.

## License

The schema and documentation in this repository are dedicated to the public domain under [CC0 1.0](LICENSE.txt).

Game data and names are the property of Fatshark.

---

*This is a fan project and is not affiliated with or endorsed by Fatshark or Games Workshop. Warhammer 40,000: Darktide is a trademark of its respective owners.*
