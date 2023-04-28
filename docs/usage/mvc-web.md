## Setup 

=== "Program.cs"

    ``` cs title="Program.cs" linenums="1" hl_lines="10-13 33-34"

    //code

    //saml2-authentication configuration
    builder.Services.AddAuthentication(sharedOptions =>
    {
        sharedOptions.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        sharedOptions.DefaultSignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        sharedOptions.DefaultChallengeScheme = Saml2Defaults.AuthenticationScheme;
    })
    .AddSaml2(options =>
    {
        //add saml2 options here
    }
    .AddCookie();

    var app = builder.Build();

    // Configure the HTTP request pipeline.
    if (!app.Environment.IsDevelopment())
    {
        app.UseExceptionHandler("/Home/Error");
        // The default HSTS value is 30 days. 
        // You may want to change this for production scenarios, 
        // see https://aka.ms/aspnetcore-hsts.
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();

    app.UseRouting();

    app.UseAuthentication();
    app.UseAuthorization();

    app.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");

    app.Run();

    ```

=== "Controller.cs"

    ``` cs title="AccountController.cs" linenums="1"

    [Authorize]
    public class AccountController : Controller
    {
        private readonly ILogger<AccountController> logger;

        public AccountController(ILogger<AccountController> logger)
        {
            this.logger = logger;
        }

        [HttpPost]
        [AllowAnonymous]
        public ActionResult ExternalLogin(string provider, string returnUrl)
        {
            // Request a redirect to the external login provider.
            if (returnUrl == null || Url.IsLocalUrl(returnUrl))
            {
                // Request a redirect to the external login provider.
                var redirectUrl = Url.Action("ExternalLoginCallback", 
                "Account", new { ReturnUrl = returnUrl });
                var properties = new AuthenticationProperties { RedirectUri = redirectUrl };

                //add the following if you are using asp.identity,
                //signInManager uses the item 'LoginProviderKey'
                properties.Items["LoginProviderKey"] = provider;
                return Challenge(properties, provider);
            }
            return RedirectToAction("Index", "Home");
        }

        [HttpGet]
        [AllowAnonymous]
        public IActionResult ExternalLoginCallback()
        {
            return RedirectToAction("Index", "Home");
        }

        [HttpPost]
        public async Task Logout()
        {
            var result = await HttpContext.AuthenticateAsync();
            var properties = result.Properties;
            var provider = properties.Items[".AuthScheme"];
            await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
            await HttpContext.SignOutAsync(provider, properties);
        }
    }

    ```

=== "Layout View"

    ``` html title="Layout.cshtml" hl_lines="28-46"

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>@ViewData["Title"]</title>
        <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
        <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />  
    </head>
    <body>
        <header>
            <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
                <div class="container-fluid">
                    <a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">Mvc.Post.ArtifactBinding</a>
                    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                            aria-expanded="false" aria-label="Toggle navigation">
                        <span class="navbar-toggler-icon"></span>
                    </button>
                    <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
                        <ul class="navbar-nav flex-grow-1">
                            <li class="nav-item">
                                <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Index">Home</a>
                            </li>
                            <li class="nav-item">
                                <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
                            </li>
                        </ul>
                        @if (!Context.User.Identity.IsAuthenticated)
                        {
                            <form class="d-flex" asp-area="" asp-controller="Account" asp-action="ExternalLogin" method="post">
                                <button type="submit" class="btn btn-primary btn-lg" name="provider" value="Saml2">Login</button>
                            </form>
                        }
                        else
                        {
                            <div class="d-sm-inline-flex justify-content-between">
                                <ul class="navbar-nav flex-grow-1 px-2">
                                    <li class="nav-item">
                                        <span class="fw-bold">@User.Identity.Name</span>
                                    <li class="nav-item">
                                </ul>
                            </div>
                            <form class="d-flex" asp-controller="account" asp-action="logout" method="post">
                                <button type="submit" class="btn btn-primary btn-lg" name="provider" value="Saml2">Logout</button>
                            </form>
                        }
                    </div>
                </div>
            </nav>
        </header>
        <div class="container">
            <main role="main" class="pb-3">
                @RenderBody()
            </main>
        </div>

        <footer class="border-top footer text-muted">
            <div class="container">
                &copy; 2023 - Artifact.PostBinding - <a asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
            </div>
        </footer>
        <script src="~/lib/jquery/dist/jquery.min.js"></script>
        <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
        <script src="~/js/site.js" asp-append-version="true"></script>
        @await RenderSectionAsync("Scripts", required: false)
    </body>
    </html>

    ```