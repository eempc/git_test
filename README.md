9. Update the razor pages and associated cs files:
    * Manage/Index.cshtml changes required:

```diff
<div class="form-group">
+   <div class="form-group">
+       <label asp-for="Input.Name"></label>
+       <input asp-for="Input.Name" class="form-control" />
+   </div>
+   <div class="form-group">
+       <label asp-for="Input.DateOfBirth"></label>
+       <input asp-for="Input.DateOfBirth" class="form-control" />
+   </div>

    <label asp-for="Input.PhoneNumber"></label>
    <input asp-for="Input.PhoneNumber" class="form-control" />
    <span asp-validation-for="Input.PhoneNumber" class="text-danger"></span>
</div>
```

* Manage/Index.cshtml.cs changes required:

```diff
public class InputModel {
+   [Required]
+   [DataType(DataType.Text)]
+   [Display(Name = "Full name")]
+   public string Name { get; set; }

+   [Required]
+   [DataType(DataType.Date)]
+   [Display(Name = "Birth date")]
+   public DateTime DateOfBirth { get; set; }

    [Phone]
    [Display(Name = "Phone number")]
    public string PhoneNumber { get; set; }
}

private async Task LoadAsync(WebApplication2User user) {
    var userName = await _userManager.GetUserNameAsync(user);
    var phoneNumber = await _userManager.GetPhoneNumberAsync(user);

    Username = userName;

    Input = new InputModel {
+       Name = user.Name,
+       DateOfBirth = user.DateOfBirth,
        PhoneNumber = phoneNumber
    };
}

public async Task<IActionResult> OnPostAsync() {
    var user = await _userManager.GetUserAsync(User);
    if (user == null) {
        return NotFound($"Unable to load user with ID '{_userManager.GetUserId(User)}'.");
    }

    if (!ModelState.IsValid) {
        await LoadAsync(user);
        return Page();
    }

+   if (Input.Name != user.Name) user.Name = Input.Name;
+   if (Input.DateOfBirth != user.DateOfBirth) user.DateOfBirth = Input.DateOfBirth;

    var phoneNumber = await _userManager.GetPhoneNumberAsync(user);
    if (Input.PhoneNumber != phoneNumber) {
        var setPhoneResult = await _userManager.SetPhoneNumberAsync(user, Input.PhoneNumber);
        if (!setPhoneResult.Succeeded) {
            var userId = await _userManager.GetUserIdAsync(user);
            throw new InvalidOperationException($"Unexpected error occurred setting phone number for user with ID '{userId}'.");
        }
    }

+   await _userManager.UpdateAsync(user);

    await _signInManager.RefreshSignInAsync(user);
    StatusMessage = "Your profile has been updated";
    return RedirectToPage();
}
```

    * Register.cshtml changes required:

```diff
<form asp-route-returnUrl="@Model.ReturnUrl" method="post">
    <h4>Create a new account.</h4>
    <hr />
    <div asp-validation-summary="All" class="text-danger"></div>
+   <div class="form-group">
+       <label asp-for="Input.Name"></label>
+       <input asp-for="Input.Name" class="form-control" />
+       <span asp-validation-for="Input.Name" class="text-danger"></span>
+   </div>
+   <div class="form-group">
+       <label asp-for="Input.DateOfBirth"></label>
+       <input asp-for="Input.DateOfBirth" class="form-control" />
+       <span asp-validation-for="Input.DateOfBirth" class="text-danger"></span>
+   </div>

    Etc...
</form>
```

    * Register .cs file changes:

```diff
public class InputModel {
+   [Required]
+   [DataType(DataType.Text)]
+   [Display(Name = "Full name")]
+   public string Name { get; set; }

+   [Required]
+   [Display(Name = "Birth Date")]
+   [DataType(DataType.Date)]
+   public DateTime DateOfBirth { get; set; }

    etc...
}

public async Task<IActionResult> OnPostAsync(string returnUrl = null) {
    returnUrl = returnUrl ?? Url.Content("~/");
    ExternalLogins = (await _signInManager.GetExternalAuthenticationSchemesAsync()).ToList();
    if (ModelState.IsValid) {
        var user = new WebApplication2User {
+           Name = Input.Name,
+           DateOfBirth = Input.DateOfBirth,
            UserName = Input.Email,
            Email = Input.Email
        };
    }
    etc...
}
```
