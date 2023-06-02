## Week 2

## At a Glance

- I have a well-cleaned database, which may have thrown out a few unique objects
- I created a Jupyter notebook with a User Interface to sift through potential
  duplicates using NED data.
- Next steps are to get SDSS subsets of Masers and Nonmasers

### Summary
This week, I made and refined a program for detecting duplicates in data. I
started out using SQL but that was limited to very crude comparisons; I needed a
better method of checking each potntial duplicate manually. In a database of
~4000 objects, there are around 100 potential duplicate pairs, which is
managable for manual duplicate removal, if a good UI is provided. I made an
object comparison UI, using NED to help make the decision.

### Duplication removal program step-by-step explanation
The process invloves finding pairs of objects in a single database that are
within a "suspect radius" of each other. The radius I chose was 1 arcsecond.

Using Astropy's "search around sky" function, pairs of objects are found. Pairs
that are duplicates of other pairs are removed (Astropy counts obj1 compared
with obj2 as a different match from obj2 compared with obj1, because the
function typically is used on seperate data sets)... my code removes the second
pair by comparing the angular separation and removing duplicates of the same angular separation from the duplicates list.

Then for each potential duplicate in the duplicates list, a reference table is
made for each object in each duplicate pair. For the reference, if True then the
object will be kept, if False then the object will not be kept. The User
interface allows manipulation of these flags.

For each duplicate pair, Ned is queried by name. If the same, and only one, name
match is found for both objects, then the objects are assumed to be a duplicate,
and the user is prompted to choose whichever they like. If the "Ask_for_all"
config flag is False, then the first object is automatically chosen. In my
cleansing process of all_surveyed, I kept "Ask_for_all" True, and simply chse
whichever object most closely matched Ned's object Name.

If the NED name query is unsuccessful, as often happens, then NED is queried by
sky location and the search radius being employed in the script for each object,
and these results, along witht he name search results, are displyed to the user.
Often the location search returns exactly 1 result; then it is safe to assume
that the pair is a duplicate. The user should choose the object with the closest
matching name. This is not automated, but could be in the future if Ask_for_all
is False.

The third, rare, possibliity is that there are many objects returned by the
location search. Ned's location search has the potential to return many
references to the same or similar object, though this is very rare. It happened
about twice in the data processing If I remember correctly. I just chose one
object to keep.

Very rarely, the objects seem to have different names and the angular separation
is relatively high. I may have kept both objects in that case (It happened once)

### Cleansing errors
There are two potential sources of error in the program. The first comes from
the rare situation when there are 3 nearby objects that are each within the
search radius of one or both of the others. The program creates separate
"potential duplicate" entries for each pair of objects. This happened about 2 or
3 times in the cleansing process. In such a situation it is possible to throw
out separate objects, or keep a duplicate.

The second source of error is that astropy's "search around sky" function
appears to not be entirely accurate; running the process again produces a
significant number of new matches (12 the first time, then 6, then 4). This
could be cleaning up the previous error source, but it seemed to be entirely new
objects on the 3rd and 4th cleansing attempts.

It would be possible to re-implement astropy's search-around_sky function using
the small angle approximation of the angular separation formula, myself. While I
have not done this, it would be interesting to see if that produces more
complete removal results than astropy's function.


## Cleansing results.

I ran the cleansing script 4 times on william's pre-cleansed data.

## Maser / nonmaser tables, python sql query creation script

I made the non-maser table, using a search radius of 5 arcsec. If there was an
object in "all_surveyed" that was within 5 arcsec of an object in the masers
table, then it was assumed to be the same object and was excluded from the new
non-masers database. 

a radius of 
6" yeilded 4775 non-masers.
5" yielded 4779 non-masers.
3" -> 4781
1" -> 4792

5" was assumed to be good enough. Strangely, returning the objects eliminated
between a radius of 5" and 6" only returned 3, not 4 objects.

I made a python script to build an SQL query to remove masers based on angular
separation. It can be found in the SQL repository.


### Next steps

Next is to crossmatch the SDSS database with my cleansed data, and create a
subset of All_surveyed_(my cleansed version), and of all_surveyed_(william's
cleansed version), and of the masers, which are a subset of the SDSS database.
Save their best object ids as a new field in the databases. Only these subsets
can be compared with the CLASS database, which is itself a subset of SDSS


