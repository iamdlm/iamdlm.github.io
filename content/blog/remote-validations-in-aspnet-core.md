---
title: "Remote Validations in ASP.NET Core"
date: 2023-05-12T20:45:26+01:00
draft: true
---
# Remote Validations in ASP.NET Core
In modern web development, validating user input is a critical aspect of creating robust and reliable applications. When it comes to remote validations, C# Razor Pages provide a powerful mechanism to validate user input without making a round trip to the server. In this blog post, we will dive into the concept of remote validations and demonstrate how to implement them in C# Razor Pages using a simple example.

Let's consider the `Student` class that represents student information. We want to validate the enrollment date to ensure it is not in the future. To achieve this, we need to apply remote validation to the `EnrollmentDate` property of the `Student` class.

{{< highlight csharp >}}
public class Student
{
    // Properties and other attributes...

    [PageRemote(
        ErrorMessage = "Date cannot be in the future.",
        HttpMethod = "post",
        PageHandler = "ValidateEnrollmentDate"
    )]
    [DataType(DataType.Date)]
    [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
    [Display(Name = "Enrollment Date")]
    public DateTime EnrollmentDate { get; set; }

    // Other properties and methods...
}
{{< /highlight >}}

In the Student class, we apply the `[PageRemote]` attribute to the `EnrollmentDate` property. This attribute configures the remote validation behavior by specifying the error message, HTTP method, and page handler method to invoke.

In the corresponding `Create.cshtml.cs` file, we add a page handler method called `OnPostValidateEnrollmentDate`. This method performs the server-side validation logic for the `EnrollmentDate` property.

{{< highlight csharp >}}
public JsonResult OnPostValidateEnrollmentDate()
{
    bool result = Student.EnrollmentDate < DateTime.UtcNow;

    return new JsonResult(result);
}
{{< /highlight >}}

At this stage, if we try to enter a future date in the `EnrollmentDate` field, the remote validation will fail with a 400 Bad Request.
That's because the anti-forgery token is not included in the AJAX request. To fix this, we need to add the `data-val-remote-additionalfields` attribute to the input field associated with the `EnrollmentDate` property in the `Create.cshtml` file.

{{< highlight html >}}
<input asp-for="Student.EnrollmentDate" class="form-control" data-val-remote-additionalfields="__RequestVerificationToken" />
{{< /highlight >}}