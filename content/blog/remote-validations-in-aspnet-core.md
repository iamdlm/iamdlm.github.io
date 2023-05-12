---
title: "Remote Validations in ASP.NET Core with FluentValidation"
date: 2023-05-12T20:45:26+01:00
draft: false
---
# Remote Validations in ASP.NET Core
In modern web development, validating user input is a critical aspect of creating robust and reliable applications. When it comes to remote validations, ASP.NET provides a powerful mechanism to validate user input without making a round trip to the server. In this blog post, we will dive into the concept of remote validations and demonstrate how to implement them in C# Razor Pages using a simple example (source code [here](https://github.com/iamdlm/remote-validations)).

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

Now, if we try to enter a future date in the `EnrollmentDate` field, the remote validation will succeed and the form will be submitted successfully.

## Using FluentValidation for Remote Validations
[FluentValidation](https://docs.fluentvalidation.net/en/latest/) is a popular validation library in the .NET ecosystem that provides a fluent and expressive API for defining validation rules. By leveraging FluentValidation, we can seamlessly integrate remote validations into our C# Razor Pages application.

For this example, we'll use the class called `Instructor` that represents instructor information. We want to validate the hire date to ensure it is not in the future. To accomplish this, we will combine the power of the `[PageRemote]` attribute with FluentValidation.

{{< highlight csharp >}}
public class Instructor
{
    // Properties and other attributes...

    [PageRemote(
        ErrorMessage = "Date cannot be in the future.",
        HttpMethod = "post",
        PageHandler = "ValidateHireDate"
    )]
    [DataType(DataType.Date)]
    [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
    [Display(Name = "Hire Date")]
    public DateTime HireDate { get; set; }

    // Other properties and methods...
}
{{< /highlight >}}

Next, create a `InstructorValidator` class that inherits from `AbstractValidator<Instructor>`. Inside this class, we define the validation rules for the `Instructor` object using the FluentValidation API.

{{< highlight csharp >}}
public class InstructorValidator : AbstractValidator<Instructor>
{
    public InstructorValidator()
    {
        RuleFor(i => i.LastName).NotNull().Length(1, 50);
        RuleFor(i => i.FirstMidName).NotNull().Length(1, 50);
        // HireDate should not be null nor in the future.
        RuleFor(i => i.HireDate).NotNull().LessThan(DateTime.UtcNow);
    }
}
{{< /highlight >}}

In the `Create.cshtml.cs` file, we define the `OnPostValidateHireDate` page handler method. This method receives an instance of the `Instructor` object and performs the server-side validation logic. We create a validation context and pass it to the `InstructorValidator` class, which is responsible for validating the object using FluentValidation. The result is then returned as a `JsonResult`.

{{< highlight csharp >}}
public JsonResult OnPostValidateHireDate(Instructor instructor)
{
    if (instructor != null)
    {
        var properties = new[] { "HireDate" };
        var context = new ValidationContext<Instructor>(instructor, new PropertyChain(), new MemberNameValidatorSelector(properties));

        IValidator v = new InstructorValidator();
        ValidationResult result = v.Validate(context);

        return new JsonResult(result.IsValid);
    }

    return new JsonResult(false);
}
{{< /highlight >}}

Once again, don't forget to add the `data-val-remote-additionalfields` attribute to the input field associated with the `HireDate` property in the `Create.cshtml` file. This ensures that the anti-forgery token is included in the AJAX request.

Source code for this example is available [here](https://github.com/iamdlm/remote-validations).

