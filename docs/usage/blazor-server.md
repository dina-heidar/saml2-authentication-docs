
## Setup

=== "Program.cs"

    ``` cs title="Program.cs" linenums="1" hl_lines="10-13"
    
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

=== "App.razor"

    ``` html title="App.razor"

        @inject NavigationManager UriHelper

        <CascadingAuthenticationState>
            <Router AppAssembly="@typeof(Program).Assembly">
                <Found Context="routeData">
                    <AuthorizeRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)">
                        <NotAuthorized>
                            @{
                                var returnUrl = UriHelper.ToBaseRelativePath(UriHelper.Uri);
                                UriHelper.NavigateTo($"login?redirectUri={returnUrl}", forceLoad: true);
                            }
                        </NotAuthorized>
                        <Authorizing>
                            Loading...
                        </Authorizing>
                    </AuthorizeRouteView>
                </Found>
                <NotFound>
                    <LayoutView Layout="@typeof(MainLayout)">
                        <p>Sorry, there's nothing at this address.</p>
                    </LayoutView>
                </NotFound>
            </Router>
        </CascadingAuthenticationState> 
    ```

=== "MainLayout.razor"

    ``` html title="MainLayout.razor"

        @inherits LayoutComponentBase

        <div class="page">
            <div class="sidebar">
                <NavMenu />
            </div>
            <main>
                <div class="top-row px-4">
                    <AuthorizeView>
                        <Authorized>
                            <form method="get" action="logout">
                                <button type="submit" class="nav-link btn btn-link">Log out</button>
                            </form>
                        </Authorized>
                        <NotAuthorized>
                            <a href="login?redirectUri=/" class="nav-link btn btn-link">Log in</a>
                        </NotAuthorized>
                    </AuthorizeView>
                    <a href="https://docs.microsoft.com/aspnet/" target="_blank">About</a>
                </div>
                <article class="content px-4">
                    @Body
                </article>
            </main>
        </div>
    ```

=== "Index.razor"

    ``` html title="Index.razor"

        @page "/"

        <PageTitle>Index</PageTitle>

        <AuthorizeView>
            <Authorized>
                <h1>Hello @context.User.Identity.Name!</h1>

                Welcome to your new app.

                <SurveyPrompt Title="How is Blazor working for you?" />
            </Authorized>
            <NotAuthorized>
                <h1>Not Authorized, please log in</h1>
            </NotAuthorized>
        </AuthorizeView>
    ```


## Login

=== "Login.cshtml"

    ``` html title="Login.cshtml"

        @page
        @model Sample.Pages.LoginModel
        @{
        }
    ```

=== "Login.cshtml.cs"

    ``` cs title="Login.cshtml.cs"

    public class LoginModel : PageModel
    {
        public async Task OnGet(string redirectUri)
        {
            await HttpContext.ChallengeAsync("Saml2", new AuthenticationProperties { RedirectUri = redirectUri });
        }
    }

    ```

## Logout

=== "Logout.cshtml"

    ``` html title="Logout.cshtml"
    @page
    @model Sample.Pages.LogoutModel
    @{
    }
    ```

=== "Logout.cshtml.cs"

    ``` cs title="Logout.cshtml.cs"

    public class LogoutModel : PageModel
    {
        public async Task<IActionResult> OnGetAsync()
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