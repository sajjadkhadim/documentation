# documentation

User Custom Claims
https://deblokt.com/2019/09/27/05-identityserver4-adding-custom-properties-to-user/

Adding custom properties to a “User” model doesn’t mean that these custom properties are going to get exposed in id or access tokens. To expose a custom user property in tokens we need to create the “IProfileService” implementation. In the project root create a new folder called “Services” and add a new class named “ProfileService”. Open the “ProfileService.cs” and modify it like so:

```
using IdentityServer.Models;
using IdentityServer4.Extensions;
using IdentityServer4.Models;
using IdentityServer4.Services;
using Microsoft.AspNetCore.Identity;
using System;
using System.Linq;
using System.Security.Claims;
using System.Threading.Tasks;

namespace IdentityServer.Services
{
    public class ProfileService : IProfileService
    {
        private readonly IUserClaimsPrincipalFactory<ApplicationUser> _claimsFactory;
        private readonly UserManager<ApplicationUser> _userManager;

        public ProfileService(UserManager<ApplicationUser> userManager, IUserClaimsPrincipalFactory<ApplicationUser> claimsFactory)
        {
            _userManager = userManager;
            _claimsFactory = claimsFactory;
        }

        public async Task GetProfileDataAsync(ProfileDataRequestContext context)
        {
            var sub = context.Subject.GetSubjectId();
            var user = await _userManager.FindByIdAsync(sub);
            var principal = await _claimsFactory.CreateAsync(user);

            var claims = principal.Claims.ToList();
            claims = claims.Where(claim => context.RequestedClaimTypes.Contains(claim.Type)).ToList();

            // Add custom claims in token here based on user properties or any other source
            claims.Add(new Claim("employee_id", user.EmployeeId ?? string.Empty));

            context.IssuedClaims = claims;
        }

        public async Task IsActiveAsync(IsActiveContext context)
        {
            var sub = context.Subject.GetSubjectId();
            var user = await _userManager.FindByIdAsync(sub);
            context.IsActive = user != null;
        }
    }
}
```
  
All that is left to do is to add a profile service dependency injection. Open “Startup.cs” and add a scoped service at the end of the “ConfigureServices” method like so:
```
services.AddScoped<IProfileService, ProfileService>();  
```
