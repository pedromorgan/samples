Revel Validation Example
===============================

The validation app demonstrates every way that the [Validation](https://revel.github.io/manual/validation)
system can be used to good effect.

Here are some core features:-

* validation/app/
	* models
		* ['`user.go`](/validation/app/models/user.go) -  User struct and validation routine.
	* controllers
		* ['`app.go`](/validation/app/controllers/app.go) - Introduction
		* ['`sample1.go`](/validation/app/controllers/sample1.go) - Validating simple fields with error messages shown at top of page.
		* ['`sample2.go`](/validation/app/controllers/sample1.go) -  Validating simple fields with error messages shown inline.
		* ['`sample3.go`](/validation/app/controllers/sample1.go) -  Validating a struct with error messages shown inline.

