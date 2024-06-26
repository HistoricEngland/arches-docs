#########
Searching
#########

Help within the Arches application
----------------------------------
The Arches application itself comes bundled with brief guides on the use of search features. Users can open these guides by clicking on the **?** button in the top right of the search interface. The animation below illustrates activation of the search guide.

.. image:: ../images/search-help-guide.gif



Term search and negation
------------------------
Arches supports powerful features to search text. The text search can be used to match strings of text in any branch describing resource instances. One can also click on a search term for opposite effect, that is, to negate or exclude resource instances that match a given term. The animation below illustrates a text search to find resource instances containing the text “Iran” then a text search that excludes resource instances that contain the term “Iran”.

.. image:: ../images/search-term-contains-and-negate.gif


Search operators
----------------
The Arches search feature supports a number of operators that you can use to find and retrieve information. The following lists different operator types and their expected search behavior:

    **Exact**: Putting your search term within quotes executes an exact search. An exact search only finds values that match then entire search string exactly, including case. For example, if the search term is “\"Excavation Unit\"” it will find the value “Excavation Unit 4”. It will *NOT* find “... this excavation unit ...“.

    **Like**: This is a prefix based search. It will search for strings that begin with each term in the search string in any order and that there is a match for each term. It is not a "contains" search. For example, if the search term is “st bo” it will find “... stock book ...” , or “... books in stock ...”. It will *NOT* find a value with just “book” or find the value “last book” (where “last” ends in “st”).

    **Equals**: This is a complete phrase search. It will only match full words in the exact order given. For example, if the search term is “stock book” it will find the value “Knoedler Stock Book 4”. It will *NOT* find “... books in stock ...“, or “... stocking new books ...”

    **Wildcard**: This assumes the user is going to supply a search phrase exactly as they want. For example, a search for "st?ck" will find the values of “stock”, or “Stick”.
    It will *NOT* find “The car is stuck in the woods”, because the “stuck” is surrounded by other words. If you need a "contains-like" query, then you need to supply leading and trailing asterisks. So, "\*st?ck\*" *WILL* find “The car is stuck in the woods”.

In all the examples above we quote the search string for clarity. In the Arches application you wouldn’t need to put your search terms inside quotes.


Escaping search operator characters
-----------------------------------
Arches version 7.4 introduced features that allow users to use backslashes ("\\") to escape special characters used as search operators. For example, if one wanted to do a search for the string “sheep?”, where the retrieved text needs to include the question-mark "?" character, one would need to escape the "?" character so that it is **NOT** interpreted as a **Wildcard** search operator (see above). The string "sheep\\?" escapes the "?" character and would search for the string “sheep?”.


Advanced Search
---------------
Arches also has "Advanced Search" options that enable users to search with specific branches to find resource instances. This adds much more precision to a search. For example, if you have resource instances described by both a "Color" branch and a "Material" branch a simple search for the term “gold” may return some unwanted results. The Advanced Search allows you to specify that you want to search within the "Material" branch, so a search for the term “gold” will return resource instances described by the material "gold", not just the color "gold".

Advanced Search is very powerful because you can use it to combine multiple search criteria together and compose complex queries. The animation below illustrates use of the Advanced Search option to search within specific branches.

.. image:: ../images/advanced-search-example.gif