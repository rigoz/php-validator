# PHP Validator
Simple form validation PHP class for POST requests, useful when coding without a framework.
It uses associative arrays to assign $_POST keys to shorthands, which are provided as arguments to the class constructor.

1. [Featured Constraints](#featured-constraints)
2. [Usage](#usage)
3. [Callback for unique constraint](#callback-for-unique-constraint)
4. [Error Handling](#error-handling)
5. [Old input](#old-input)
6. [Extensibility](#extensibility)
7. [Demo](#demo)
8. [Requirements](#requirements)

# Featured Constraints
It supports the following constraints:
- required: checks if the field is not empty
- unique: checks if the field value is not taken through a callback function provided by yourself
- alpha: checks if the field contains only letters (extended alphabet letters like à,ò,è,ù are allowed as well
- integer: checks if the field is an integer number
- float: checks if the field is a decimal number
- email: checks if the field is a valid email address


#Usage
Validator uses a shorthand-to-field association to handle input fields, in which array keys are the shorthands, and array values are either field names, constraints or other type of data.

When instantiating Validator class, you need to provide an array of arguments, where these 4 elements must be present with the following keys:
- 'trigger': which is the field to check if POST data has been submitted, usually the name attribute of the submit button
- 'id': which is the name attribute of the record id being edited, set it to '' (empty string) if you are validating on insert instead
- 'input': which is an array where keys are field shorthands, and values are field name attributes
- 'constraints': which is an array where keys are field shorthands (the same as input), and values are constraints separated by a space

You can also provide 2 more elements to the array of arguments:
- 'uniqueCallbacks': which is an array of callback functions for unique constraint check, with keys as shorthands and values as arrays of callables
- 'defaults': which is an array of default values returned when getOldInput() has no data, where keys are shorthands and values are field values

An example of array of arguments should look like this:
```
$args = [
	'trigger' 		=> 'submit',
	'id'			=> 'user-id',
	'input' 		=> [
		'name'			=> 'user-name',
		'email'			=> 'user-email',
	],
	'constraints' 	=> [
		'name'			=> 'required alpha',
		'email'			=> 'required email unique',
	],
	'uniqueCallbacks' => [
		'email'			=> ['myClass', 'isEmailAvailable']
	],
	'defaults' => [
		'name'			=> 'John',
		'email'			=> 'john@domain.com',
	],
];
```
To validate data the method validate() must be called, which returns TRUE or FALSE accordingly

Basic example validating data:
```
// $args is the one provided above
$validator = new Validator($args);

if ($validator->hasData())
{
	if ($validator->validate())
		echo 'Data is valid';
	else
		echo 'Data is invalid';
}
```

# Callback for unique constraint
This feature is intended for database checks and calls an user-provided function which should check for uniqueness of the value in the database.
To the user-provided function will be passed two arguments, $id and $value, respectively the id of the record we are modifying (0 in case we are adding a new record) 
and the value to check.
The user-provided function should return TRUE if $value is not taken by another record, FALSE otherwise.


A basic example for a valid callback would be:
```
function isEmailAvailable($id, $value)
{
	$connection = new PDO('mysql:host=localhost;dbname=testdb;charset=utf8', 'username', 'password');
	
	$statement = $connection->prepare('SELECT email FROM users WHERE email = :email AND id != :id');
	$statement->bindValue(':email', $value, PDO::PARAM_STR);
	$statement->bindValue(':id', $id, PDO::PARAM_INT);
	
	$statement->execute();
	
	return $statement->fetchAll() == null;
}
```
# Error handling
When a field does not pass a constraint check, an error message is saved for later view.
You can get all error messages by calling the method getErrors(). It returns an associative array with field shorthands as keys and messages as values.

Example:
```
// assuming our validator is instantiated as $validator
$errors = $validator->getErrors();
echo $errors['email'];
```

# Old input
When updating an element with a form, you may want to show the user what he typed after validation occurs, especially if some fields dont pass the checks. You can do this by setting the values of input fields to the values returned by the method getOldInput(). If there is no old input to display, which basically means the user never submitted the form, getOldInput() will return custom default values specified by the 'defaults' array passed to the constructor.

Example:
```
// assuming our validator is instantiated as $validator
$input = $validator->getOldInput();
echo '<input type="text" name="user-email" value="' . $input['email'] . '" />';
```
# Extensibility
This class is written to be extensible on the validation and the error handling.

### Extending validation
You may extend validation with two methods:
- preValidation() which is called before the validation takes any action, and should not return any value
- postValidation() which is called after validation has occured and affects the final result of the validation by returning TRUE or FALSE

### Extending error handling
You may alter the error messages by implementing your own static method getErrorMessage().
Standard method looks like this:
```
public static function getErrorMessage($code)
{
	switch($code)
	{
		case 'E_MISSING_FIELD'	: return 'This field is missing from data request';
		case 'E_REQUIRED_FIELD'	: return 'This field is required';
		case 'E_UNIQUE_FIELD'	: return 'This value is already in use';
		case 'E_ALPHA_FIELD'	: return 'This field cannot contain numbers or symbols';
		case 'E_INTEGER_FIELD'	: return 'This field can only contain integer numbers';
		case 'E_FLOAT_FIELD'	: return 'This field can only contain decimal numbers';
		case 'E_EMAIL_FIELD'	: return 'This is not a valid email address';
	}
}
```
This is especially useful for multilanguage support.

# Demo
You can find a basic example at [this page](http://www.rigo7.com/projects/validator?source=git)


# Requirements
Validator requires PHP 5.3.0 to support late static binding
