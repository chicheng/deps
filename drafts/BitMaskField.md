# BitMask Field

|Meta	|Value		|
|:-----:|:----------|
|Created|2014-04-19 |
|Author |CHI Cheng	|
|Status |Draft		|

## Overview

In many cases, we require a many-to-many relation with limited set of value on one side, where ManyToManyField is overkill.

BitMaskField provides simpler database structure as well as better query performance.

## Rationale


## Implementation

BitMaskField(s) are based on Positive(Small)IntegerField and BigIntegerField.

Maximum number of options (round down to nearest multiple of 5)

|Field		|Maximum number of options		|
|:---------:|:------------------------------|
|SmallBitMaskField	| 15	|
|BitMaskField		| 30	|
|BigBitMaskField	| 60	|

Use ManyToManyField if require more.

### Definition
	
	# Reuse existing choices argument format
	DAY_OF_WEEK = (
		('monday', _('Monday')),
		('tuesday', _('Tuesday')),
		('wednesday', _('Wednesday')),
		('thursday', _('Thursday')),
		('friday', _('Friday')),
		('saturday', _('Saturday')),
		('sunday', _('Sunday')),
	)

	open_days = models.SmallBitMaskField(choices=DAY_OF_WEEK)


### Read

	if store.open_days.saturday and store.open_days.sunday:
		# store open on weekends
		pass

### Write

#### Set value

	store.open_days = Store.open_days.monday | Store.open_days.tuesday

	# equal to

	store.open_days.clear()
	store.open_days.monday = True
	store.open_days.tuesday = True

	# equal to

	# Allow many-to-many fields methods
	store.open_days.clear()
	store.open_days.add(Store.open_days.monday, Store.open_days.tuesday)

#### Add value

	store.open_days |= Store.open_days.wednesday

	# equal to

	store.open_days.wednesday = True

	# equal to

	store.open_days.add(Store.open_days.wednesday)

#### Remove value

	store.open_days &= ~Store.open_days.sunday

	# equal to

	store.open_days.sunday = False

	# equal to

	store.open_days.remove(Store.open_days.sunday)

### Query

A list of options or a single value should be passed as parameter value. 
	
	# Find stores only open on Sunday (not on other days)
	Store.objects.filter(open_days=Store.open_days.sunday)
	# SQL: open_days = 0b1000000
	
	# Find stores open on Sunday (may open on other days or not)
	Store.objects.filter(open_days__in=Store.open_days.sunday)
	# or
	Store.objects.filter(open_days__contains=Store.open_days.sunday)
	# SQL: open_days & 0b1000000 > 0


	# Find stores only open on Saturday and Sunday (not on weekdays)
	Store.objects.filter(open_days=[Store.open_days.saturday, Store.open_days.sunday])
	# SQL: open_days = 0b1100000

	# Find stores open on both Saturday and Sunday (may open on weekdays or not)
	# or call it field_name__all ?
	Store.objects.filter(open_days__contains=[Store.open_days.saturday, Store.open_days.sunday])
	# SQL: open_days & 0b1000000 > 0 AND open_days & 0b0100000 > 0

	# Find stores open on Saturday and/or Sunday (may open on weekdays or not)
	# or call it field_name__any ?
	Store.objects.filter(open_days__in=[Store.open_days.saturday, Store.open_days.sunday])
	# SQL: open_days & 0b1100000 > 0


## Related functions

### Validators

Two new validators could be added to limit number of selection.

`MinSelectionValidator` and `MaxSelectionValidator`

### Form

Reuse existing many-to-many form control.


## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license](http://creativecommons.org/publicdomain/zero/1.0/deed).
