# Darktide Weapon Database

A SQL Server database of weapon stats for *Warhammer 40,000: Darktide* — damage breakdowns, extra stats, blessings, perks and specs for every weapon.

Notice: This is my first database so you may come across things that are not 100% correct in terms of format. Feel free to send an issue if anything seems off.

> **Game version:** [1.13.0 / 2026-10-24]
> **Last updated:** [2026-10-24]

---

## Contents

| File | What it is | Use it if… |
|---|---|---|
| `Darktide_Weapons.bak` | SQL Server backup (see [Releases](../../releases)) | You have **SQL Server 2025 (17.x) or newer** |
| `csv/` | One CSV file per table | You want to open the data in Excel, Python, etc. |
| `schema/` | Table definitions only, one file per table | You just want to see the structure |

---

## Tables

| Table | Description |
|---|---|
| `DamageBreakdown` | Damage per hit vs. each enemy armour type (Unarmoured, Infested, Flak, Unyielding, Carapace, Maniac) — body, weakspot, critical and critical weakspot, low/high values |
| `ExtraStats` | Detailed weapon stats: ammo, blast radius, cleave, stagger, crit bonuses, mobility, stability, reload, etc. (low/high values) |
| `Blessings` | Available blessings per weapon with descriptions |
| `Perks` | Available perks per weapon with low/high values and units |
| `Specs` | Base weapon specs: attack speed, dodge, sprint, actions and weapon type |

---

## Installation

### Restore the .bak (SQL Server 2025+ only)

**SSMS:** right-click **Databases**, then **Restore Database… → Device**, pick `Darktide_Weapons.bak` and click **OK**.

**SQL:** first check the logical file names in the backup:
```sql
RESTORE FILELISTONLY FROM DISK = 'C:\path\to\Darktide_Weapons.bak';
```
Then restore it, moving the files into your own data folder:
```sql
RESTORE DATABASE Darktide
FROM DISK = 'C:\path\to\Darktide.bak'
WITH MOVE '[logical data name]' TO 'C:\...\MSSQL\DATA\Darktide.mdf',
     MOVE '[logical log name]'  TO 'C:\...\MSSQL\DATA\Darktide_log.ldf',
     RECOVERY;
```
On Linux, use `/var/opt/mssql/data/` as the target folder. Put the `.bak` somewhere the `mssql` user can read it.

### Option 2: CSV files

Open them in any spreadsheet program, or load them into the database of your choice. Each file has a header row with the column names.

---

## Example queries

**Weapons with explosive blessings, sorted by blast radius:**
```sql
SELECT DISTINCT
    damage.Weapon,
    damage.UnarmouredBodyHigh AS UnArmrBodyDmgHi,
    extra.BlastRadiusBlastRadiusHigh AS BlastRadHi,
    blessing.Description
FROM Darktide_Weapons.DamageBreakdown damage
JOIN Darktide_Weapons.ExtraStats extra ON extra.Weapon = damage.Weapon
JOIN Darktide_Weapons.Blessings blessing ON blessing.Weapon = damage.Weapon
WHERE blessing.Description LIKE '%explosion%'
ORDER BY BlastRadHi DESC, UnArmrBodyDmgHi DESC;
```

**Best weapons vs. Carapace armour (weakspot hits):**
```sql
SELECT Weapon, Class, CarapaceArmouredWeakspotHigh
FROM Darktide_Weapons.DamageBreakdown
ORDER BY CarapaceArmouredWeakspotHigh DESC;
```

**All blessings for one weapon:**
```sql
SELECT Blessing, Description
FROM Darktide_Weapons.Blessings
WHERE Weapon LIKE '%[weapon name]%';
```

---

## Data source

[Data copied manually from https://darktide.gameslantern.com/]

Notice: While the data has been copied by hand, the sorting of the copied data into CSV was done by Claude.ai.

## Contributing

Found a wrong value or missing weapon? [Open an issue](../../issues) or submit a pull request.

# License

The schema and documentation in this repository are dedicated to the public domain under [CC0 1.0](LICENSE.txt).

The weapon data was collected from [Games Lantern](https://darktide.gameslantern.com/) and remains subject to their [terms of use](https://gameslantern.com/terms-of-use). Game data and names are the property of Fatshark.
---

*This is a fan project and is not affiliated with or endorsed by Fatshark or Games Workshop. Warhammer 40,000: Darktide is a trademark of its respective owners.*
