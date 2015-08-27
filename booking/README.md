Revel Hotels -  International Bookingz..zzz
===========================================

This sample application demonstrates :-

* Using a database - SQLite or others 
* Configuring the Revel DB module
* Using the third party [GORP](https://github.com/coopernurse/gorp) *ORM-ish* library
* Using [Interceptors](http://revel.github.io/manual/interceptors) for checking that an user is logged in
* Using [validation](http://revel.github.io/manual/validation) and displaying inline errors


Here is a quick summary of the important files and their purpose:

* `booking/app/`
	* `models/`		- Structs and validation
		* [`booking.go`](/booking//app/models/booking.go)
		* [`hotel.go`](/booking//app/models/hotel.go)
		* [`user.go`](/booking/app/models/user.go)
	* `controllers/`
        * [`init.go`](/booking/controllers/init.go)    - Register all of the interceptors.
        * [`gorp.go`](/booking/controllers/gorp.go)    - A plugin for setting up Gorp, creating tables, and managing transactions.
        * [`app.go`](/booking/controllers/app.go)     - "Login" and "Register new user" pages
        * [`hotels.go`](/booking/controllers/hotels.go)  - Hotel searching and booking
    * `views/`
        * .. the templates..


Database Install and Setup
--------------------------------
This example uses [sqlite](https://www.sqlite.org/), but code can be easily changed for mysql, postgres, et all.

## sqlite Installation

- The booking app uses the [go-sqlite3](https://github.com/mattn/go-sqlite3) database driver, **which wraps the native C library**.
- This means that the native c code needs to be installed first

#### Install sqlite on OSX

1. Install [Homebrew](http://mxcl.github.com/homebrew/) if you don't already have it.
2. Install pkg-config and sqlite3:

```bash
brew install pkgconfig sqlite3
```

#### Install sqlite on Ubuntu

```bash
$ sudo apt-get install sqlite3 libsqlite3-dev
```

Run the Hotel :-)
=============================
```bash
revel run github.com/revel/samples/booking
```

Developer Notes
=============================

## Database and Gorp Plugin

* The [`app/controllers/gorp.go`](/booking/app/controllers/gorp.go) defines a "kind of"  *GormPlugin* 
  * The functions are called at various stages of application startup
    * eg `InitDb()` is called on startup ..

## Interceptors

* The *plugin* is then initialised in [`app/controllers/init.go`](/booking/app/controllers/init.go) file in :-

```go
func init() {
	revel.OnAppStart(Init)
	revel.InterceptMethod((*GorpController).Begin, revel.BEFORE)
	revel.InterceptMethod(Application.AddUser, revel.BEFORE)
	revel.InterceptMethod(Hotels.checkUser, revel.BEFORE)
	revel.InterceptMethod((*GorpController).Commit, revel.AFTER)
	revel.InterceptMethod((*GorpController).Rollback, revel.FINALLY)
}
```

* **`OnAppStart()`** 
    * Uses the DB module to open a SQLite in-memory database
    * Create the `User`, `Booking`, and `Hotel` tables
    * and insert some test records.
* **`BeforeRequest()`**
    * Begins a transaction and stores the Transaction on the Controller
* **`AfterRequest()`** 
    * Commits the transaction
    * or [panics](https://github.com/golang/go/wiki/PanicAndRecover) if there was an error
* **`OnException()`** 
    * Rolls back the transaction


As an example, `checkUser` looks up the username in the `session` and `redirect`s
the user to log in if they do not have a `session` cookie.

```go
func (c Hotels) checkUser() revel.Result {
	if user := c.connected(); user == nil {
		c.Flash.Error("Please log in first")
		return c.Redirect(Application.Index)
	}
	return nil
}
```

## Validation

The booking app does quite a bit of validation.

For example, here is the routine to validate a booking, from
[models/booking.go](/booking/app/models/booking.go):

```go
func (booking Booking) Validate(v *revel.Validation) {
	v.Required(booking.User)
	v.Required(booking.Hotel)
	v.Required(booking.CheckInDate)
	v.Required(booking.CheckOutDate)

	v.Match(b.CardNumber, regexp.MustCompile(`\d{16}`)).
		Message("Credit card number must be numeric and 16 digits")

	v.Check(booking.NameOnCard,
		revel.Required{},
		revel.MinSize{3},
		revel.MaxSize{70},
	)
}
```

Revel applies the validation and records errors using the name of the
validated variable, unless overridden.  

For example, `booking.CheckInDate` is
required; if it evaluates to the zero date, Revel stores a `ValidationError` in
the validation context under the key "booking.CheckInDate".

Subsequently, the  [Hotels/Book.html](/booking/app/views/Hotels/Book.html)
template can easily access them using the [`field`](http://revel.github.io/manual/templates.html#field) helper:

```
{{with $field := field "booking.CheckInDate" .}}
<p class="{{$field.ErrorClass}}">
    <strong>Check In Date:</strong>
    <input type="text" size="10" name="{{$field.Name}}" class="datepicker" value="{{$field.Flash}}">
    * <span class="error">{{$field.Error}}</span>
ss</p>
```

The [`field`]http://revel.github.io/manual/templates.html#field) template helper looks for errors in the validation context, using
the field name as the key.
