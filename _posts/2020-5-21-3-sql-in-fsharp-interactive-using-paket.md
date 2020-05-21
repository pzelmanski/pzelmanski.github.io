---
layout: post
title: F# | Running Sql commands from F# interactive using paket
categories: "FSharp F# jetbrains rider interactive fsx"
---

Running Sql commands directly from F# interactive within rider is a bit problematic. I've spent a few hours trying to do this, and I feel like it's way more complicated than it could be.

# How to use SqlClient in F# interactive using packet and rider on .NET Core 3.1

Prerequisities:
1. Locally installed SQL Server.
    * Doesn't need to be installed locally, but you need any SQL Server instance
2. Some data in sql server
    * Not totally required, but I used this to test whether running sql query works or not. I created some dummy table with dummy data with this script:

```sql
create table test1(
		id int primary key,
		name varchar(50))

insert into test1 values(1, 'n1')
insert into test1 values(2, 'n2')
insert into test1 values(3, 'n3')
insert into test1 values(4, 'n4')

select * from test1
```

And basically that's all around database.

# Paket commands

1. Create empty solution
2. Add .NET Core F# 3.1 Class Library named `MainProject`
3. Run commands:
    * `dotnet new tool-manifest`
    * `dotnet tool install paket`
    * `dotnet tool restore`
    * `dotnet paket init`
    * `dotnet paket add FSharp.Data.SqlClient --project PaketTest`
    * `dotnet paket install`
    * `dotnet paket generate-load-scripts --framework net45`

As you can see, the last command points at .NET 4.5 framework. 

But how is that? It was supposed to be .NET Core!

Yeah, so I haven't managet to get it running with even .NET standard 2.0. If you know any better way, I'd love to hear it! **For this to work, you need to change F# interactive target from .NET Core 3.1, in my case I needed to uncheck `Choose F# interactive automatically` and change `F# interactive tool` from `.NET Core SDK 3.1.201` to `Visual Studio 2019 Community`**
![test coverage](/images/3-sql-fsharp-interactive/fsharp-interactive-net-version.png)
Also, it's recommended to add this directories to .gitignore:
```
/packet-files/
/.packet/
```
Add script to your project, named however you like (I named DbTest.fsx)
And finally, fill your `.fsx` script with F# code:

```Fsharp
#load "../.paket/load/net45/FSharp.Data.SqlClient.fsx"

open FSharp.Data

[<Literal>]
let connStr = "Server=.;Initial catalog=DATABASE_NAME;Trusted_Connection=true"

let GetData() = do
    use cmd = new SqlCommandProvider<"SELECT TOP(@topN) * from test1" , connStr>(connStr)
    cmd.Execute(topN = 3L) |> printfn "%A";;
```

That's all. When I call GetData(), I have this result:
```
seq
  [{ id = 1; name = Some "n1" }; { id = 2; name = Some "n2" };
   { id = 3; name = Some "n3" }]
val it : unit = ()
```

# Repository with working solution

You can find test solution with working code on my GitHub, [here](https://github.com/pzelmanski/SqlClient-FSharp-interactive).
Just run paket commands, open `DbTest.fsx` and run in F# interactive. Maybe you'd also like to change connection string a bit.
