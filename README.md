# BlazorApp

This is my first Blazor Server app.  
It has login and signup using Identity and it also saves data with EF Core in a SQLite database.  

---

## How it works (what I understood)

- When you run the app, you can click **Register** to make a new account.  
- The account info is stored in a file database called `app.db` inside the `Data` folder.  
- When you log in, the app knows who you are because it checks the cookie and Identity system.  
- On pages where I put `[Authorize]`, only logged in users can see them.  
- There is also `<AuthorizeView>` in Blazor which shows different content depending if the user is logged in or not.  

Example of `<AuthorizeView>` in a Razor component:

```
<AuthorizeView>
    <Authorized>
        <p>Hello, @context.User.Identity.Name!</p>
    </Authorized>
    <NotAuthorized>
        <p>Please log in to see this page.</p>
    </NotAuthorized>
</AuthorizeView>
```

<Authorized> content is only shown if the user is logged in.

<NotAuthorized> content is shown if the user is not logged in.

So basically:

Not logged in → you only see normal public pages.

Logged in → you can see your name at the top and also private pages.

Entity Framework (EF) part
The project uses EF Core with SQLite.

The connection string is in appsettings.json:

```
"ConnectionStrings": {
  "DefaultConnection": "Data Source=Data/app.db;Cache=Shared"
}
```

To update the database, I had to run:
```
dotnet ef migrations add InitialCreate
dotnet ef database update
```

This created the app.db file and tables for Identity (like AspNetUsers, AspNetRoles, etc).

Adding your own data (example)
I also tried creating my own table and saving data:

Create a simple model:

```
public class Note
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
}
```

Add it to the DbContext:

```
public DbSet<Note> Notes { get; set; }
```

Add a migration and update the database:

```
dotnet ef migrations add AddNoteTable
dotnet ef database update
```

Running the app
Clone this repo.

Open terminal in the project folder.

Run:

Copy code
dotnet run
Open browser at https://localhost:5001 (or whatever it shows).

## Weather Page

The Weather page shows a list of weather forecasts from the database instead of random data.

### How it works

- The page uses **Entity Framework Core** to fetch data from the `WeatherForecasts` table in `app.db`.
- We inject the `ApplicationDbContext` into the Razor page with:

```
@inject ApplicationDbContext DbContext
```

- The page then fetches the data asynchronously using:

```
forecasts = await DbContext.WeatherForecasts
    .OrderBy(w => w.Date)
    .ToListAsync();
```

- While the data is loading, it shows a Loading... message.

- Once the data is loaded, it displays a table with:

    - Date

    - Temperature in Celsius

    - Temperature in Fahrenheit

Summary

If no data is available, it shows No weather data available.

Code snippet
```
@if (forecasts == null)
{
    <p><em>Loading...</em></p>
}
else if (!forecasts.Any())
{
    <p><em>No weather data available.</em></p>
}
else
{
    <table class="table">
        <thead>
            <tr>
                <th>Date</th>
                <th>Temp. (C)</th>
                <th>Temp. (F)</th>
                <th>Summary</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var forecast in forecasts)
            {
                <tr>
                    <td>@forecast.Date.ToShortDateString()</td>
                    <td>@forecast.TemperatureC</td>
                    <td>@forecast.TemperatureF</td>
                    <td>@forecast.Summary</td>
                </tr>
            }
        </tbody>
    </table>
}
```

Notes

- Make sure the WeatherForecasts table exists in your SQLite database.
- You can add new records using EF Core migrations or directly seeding them in Program.cs.
- This page demonstrates fetching and displaying data from a real database in Blazor Server, which is a step up from the sample random data example.