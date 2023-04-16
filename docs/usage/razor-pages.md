## Program

``` cs title="Program.cs" linenums="1" hl_lines="18"
//code

 var app = builder.Build();

    // Configure the HTTP request pipeline.
    if (!app.Environment.IsDevelopment())
    {
        app.UseExceptionHandler("/Error");
        // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();

    app.UseRouting();

    app.UseAuthorization();

    app.MapRazorPages();

    app.Run();
```


## Pages

``` html title="Index.cshtml"
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}

<div class="text-center">
    @if (User.Identity.IsAuthenticated)
    {
        <h1 class="display-4">Welcome @User.Identity.Name!</h1>
    }
</div>
```



``` cs title="Index.cshtml.cs"

public class IndexModel : PageModel
{
    private readonly ILogger<IndexModel> _logger;

    public IndexModel(ILogger<IndexModel> logger)
    {
        _logger = logger;
    }

    public void OnGet()
    {

    }
}

```



``` html title="Login.cshtml"
@page
@model Sample.Pages.LoginModel
@{
}
```



``` cs title="Login.cshtml.cs"

public class LoginModel : PageModel
{
    public async Task OnPost(string redirectUri)
    {
        await HttpContext.ChallengeAsync("Saml2", new AuthenticationProperties { RedirectUri = redirectUri });
    }
}

```



``` html title="Logout.cshtml"
@page
@model Sample.Pages.LogoutModel
@{
}
```

``` cs title="Login.cshtml.cs"

public class LogoutModel : PageModel
{
    public async Task<IActionResult> OnPostAsync()
    {
        var result = await HttpContext.AuthenticateAsync();
        var properties = result.Properties;
        var provider = properties.Items[".AuthScheme"];
        await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
        await HttpContext.SignOutAsync(provider, properties);

        return Redirect("/");
    }
}

```