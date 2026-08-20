WP Schema — Project Context
PROJECT: WP Schema

OWNER:
Funnelwheel

PURPOSE:
Build a schema-first content modeling engine for WordPress.

VISION:
WP Schema should bring the content modeling capabilities of modern headless
CMS platforms such as Payload, Sanity, and Hygraph to WordPress while
remaining native and familiar to WordPress developers.

WP Schema is NOT primarily an ORM.

It is a CONTENT MODELING / SCHEMA ENGINE for WordPress.

CORE IDEA:

    Schema
       ↓
    Content Model
       ↓
    Fields / Relations / Validation
       ↓
    Persistence
       ↓
    Query / Mutation APIs

WordPress should remain the developer-facing ecosystem and mental model,
while WP Schema provides a modern structured-content layer underneath.


==================================================
NAMING
==================================================

PROJECT NAME:

    WP Schema

Repository:

    funnelwheel/wp-schema

Primary PHP concepts:

    WP_Schema
    WP_Model
    WP_Model_Type
    WP_Model_Query

The API should feel native to WordPress.

Avoid branding the public API around Funnelwheel.

Avoid calling the project an "ORM".

Preferred description:

    "A schema-first content modeling engine for WordPress."

Alternative:

    "Structured content schemas and models for WordPress."


==================================================
LONG-TERM VISION
==================================================

WP Schema should eventually support:

    - Content models
    - Typed fields
    - Relations
    - Validation
    - Indexes
    - Database schema generation
    - Database migrations
    - Drafts
    - Publishing
    - Versions
    - Localization
    - Blocks / Components
    - Rich text
    - Media
    - REST API generation
    - GraphQL API generation
    - Type generation
    - Admin UI metadata
    - Permissions
    - Webhooks
    - Search

The schema should become the source of truth.

Example:

    Content Schema
          ↓
       Schema AST
          ↓
    ┌─────┼─────────┐
    ↓     ↓         ↓
    SQL  REST    GraphQL
    ↓
 Database


==================================================
CORE ARCHITECTURAL PRINCIPLE
==================================================

The following separation must be preserved:

    SCHEMA
        describes what content is

    MODEL
        represents one content item

    MODEL TYPE
        describes a collection/content type

    REPOSITORY
        persists and retrieves models

    QUERY
        describes how models are retrieved

    QUERY BUILDER
        converts queries into safe database operations

    DATABASE
        implements persistence

Never mix these responsibilities unnecessarily.


==================================================
PUBLIC API
==================================================

The initial user-facing API should be intentionally small.

Seven primary functions:

    register_model_type();

    get_model_type();

    model_type_exists();

    get_model();

    get_models();

    save_model();

    delete_model();


These functions are facades.

Business logic should live in the underlying classes.


==================================================
REGISTER MODEL
==================================================

Example:

    register_model_type('book', [
        'label' => 'Books',

        'fields' => [
            'title' => [
                'type'     => 'text',
                'required' => true,
            ],

            'slug' => [
                'type'   => 'slug',
                'unique' => true,
            ],

            'description' => [
                'type' => 'textarea',
            ],

            'published_at' => [
                'type' => 'datetime',
            ],

            'author' => [
                'type'  => 'relation',
                'model' => 'author',
            ],
        ],
    ]);


IMPORTANT:

The public schema describes CONTENT, not SQL.

Use:

    fields

rather than exposing:

    columns

as the primary public abstraction.

The database representation is an implementation detail.


==================================================
PUBLIC API EXAMPLES
==================================================

REGISTER:

    register_model_type('book', [
        'label' => 'Books',

        'fields' => [
            'title' => [
                'type' => 'text',
                'required' => true,
            ],

            'slug' => [
                'type' => 'slug',
                'unique' => true,
            ],
        ],
    ]);


GET MODEL TYPE:

    $book = get_model_type('book');

    $book->name();
    $book->label();
    $book->fields();
    $book->field('title');


CHECK MODEL TYPE:

    if (model_type_exists('book')) {
        // registered
    }


GET ONE MODEL:

    $book = get_model('book', 123);

    $book->id();
    $book->get('title');
    $book->set('title', 'New title');
    $book->save();
    $book->delete();


GET MANY:

    $books = get_models('book');

    $books = get_models('book', [
        'limit'   => 20,
        'orderby' => 'published_at',
        'order'   => 'DESC',
    ]);


FILTER:

    $books = get_models('book', [
        'where' => [
            'status' => 'published',
        ],
    ]);


RELATIONS:

    $book = get_model('book', 123);

    $author = $book->get('author');

    echo $author->get('name');


CREATE:

    $book = save_model('book', [
        'title' => 'The Great Gatsby',
        'slug'  => 'the-great-gatsby',
    ]);


UPDATE:

    $book = save_model('book', [
        'id'    => 123,
        'title' => 'Updated title',
    ]);


DELETE:

    delete_model('book', 123);


==================================================
CORE CLASSES
==================================================

Initial foundational classes:

    WP_Schema
    WP_Schema_Field
    WP_Model_Type
    WP_Model
    WP_Model_Repository
    WP_Model_Query


Later:

    WP_Schema_Registry
    WP_Schema_Validator
    WP_Schema_Compiler
    WP_Schema_Migrator
    WP_Model_Factory
    WP_Model_Relation
    WP_Model_Query_Builder
    WP_Database_Schema


==================================================
WP_SCHEMA
==================================================

Conceptual API:

    class WP_Schema {

        public static function register($name, $definition);

        public static function get($name);

        public static function has($name);

        public static function all();

        public static function unregister($name);
    }

WP_Schema represents the content schema system.

It should not directly contain SQL logic.


==================================================
WP_SCHEMA_FIELD
==================================================

Fields are first-class objects.

Conceptual API:

    abstract class WP_Schema_Field {

        public function name();

        public function type();

        public function required();

        public function nullable();

        public function default();

        public function validate($value);

        public function serialize($value);

        public function deserialize($value);

        public function database_definition();
    }


Potential field types:

    text
    textarea
    richtext
    number
    boolean
    date
    datetime
    email
    url
    slug
    json
    media
    relation
    object
    array
    block


Do not create every field type prematurely.


==================================================
WP_MODEL_TYPE
==================================================

Conceptually equivalent to WP_Post_Type.

It describes a registered content type.

    class WP_Model_Type {

        public function name();

        public function label();

        public function schema();

        public function fields();

        public function field($name);

        public function relations();

        public function repository();

        public function supports($feature);

        public function capabilities();

        public function is_registered();
    }


IMPORTANT:

WP_Model_Type should remain familiar to WordPress developers.

It can borrow useful architectural ideas from WP_Post_Type.

However, it should not blindly inherit storage assumptions from WP_Post_Type.


==================================================
WP_MODEL
==================================================

Represents a single content item.

Conceptually similar to WP_Post.

    class WP_Model {

        public function id();

        public function type();

        public function get($field);

        public function set($field, $value);

        public function has($field);

        public function all();

        public function save();

        public function delete();

        public function exists();

        public function validate();

        public function relation($name);

        public function to_array();
    }


The model should NOT know SQL.

Persistence should be delegated to a repository.


==================================================
WP_MODEL_REPOSITORY
==================================================

Responsible for persistence.

    class WP_Model_Repository {

        public function find($id);

        public function find_many($args = []);

        public function insert($data);

        public function update($id, $data);

        public function delete($id);

        public function exists($id);

        public function count($args = []);
    }


Flow:

    WP_Model
        ↓
    WP_Model_Repository
        ↓
    Database


==================================================
WP_MODEL_QUERY
==================================================

Equivalent conceptually to WP_Query.

Example:

    $books = new WP_Model_Query([
        'model_type' => 'book',
    ]);

    $books
        ->where('status', '=', 'published')
        ->orderby('published_at', 'DESC')
        ->limit(20)
        ->get();


Conceptual API:

    class WP_Model_Query {

        public function __construct($args = []);

        public function where($field, $operator, $value);

        public function where_in($field, $values);

        public function where_null($field);

        public function orderby($field, $direction = 'ASC');

        public function limit($limit);

        public function offset($offset);

        public function with($relation);

        public function get();

        public function first();

        public function find($id);

        public function count();
    }


==================================================
QUERY SECURITY
==================================================

NEVER directly interpolate untrusted values into SQL.

Flow:

    User input
        ↓
    WP_Model_Query
        ↓
    Query Builder
        ↓
    Parameterized SQL
        ↓
    $wpdb


Identifiers such as:

    field names
    table names
    order directions
    operators

must be validated or allow-listed.

Do not assume $wpdb->get_results() makes an already-built SQL string safe.


==================================================
QUERY BUILDER
==================================================

Internal implementation.

    class WP_Model_Query_Builder {

        public function select($fields);

        public function from($table);

        public function where($condition, $value);

        public function join($table, $condition);

        public function orderby($field, $direction);

        public function limit($limit);

        public function offset($offset);

        public function bindings();

        public function to_sql();
    }


Query Builder should not be the primary public API.


==================================================
SCHEMA COMPILER
==================================================

The schema should not immediately become SQL.

Flow:

    PHP Definition
        ↓
    Schema
        ↓
    Schema AST
        ↓
    Compiler
        ↓
    Database representation


Conceptual:

    class WP_Schema_Compiler {

        public function compile($schema);

        public function compile_field($field);

        public function compile_relation($relation);

        public function compile_indexes($schema);

        public function compile_constraints($schema);
    }


The Schema AST should eventually be reusable by:

    SQL
    REST
    GraphQL
    Admin
    Validation
    Type generation


==================================================
DATABASE SCHEMA
==================================================

Database concerns belong here.

    class WP_Database_Schema {

        public function create($schema);

        public function update($schema);

        public function drop($schema);

        public function exists($table);

        public function columns($table);

        public function indexes($table);

        public function constraints($table);
    }


Do not make the public content schema SQL-specific.


==================================================
MIGRATIONS
==================================================

NEVER automatically destroy data simply because a PHP schema changed.

Avoid:

    remove field from PHP
        ↓
    automatically DROP COLUMN


Instead:

    Schema version
        ↓
    Schema diff
        ↓
    Migration plan
        ↓
    Explicit migration


Conceptual:

    class WP_Schema_Migrator {

        public function plan($schema);

        public function apply($plan);

        public function rollback($version);

        public function status();
    }


Potential WP CLI:

    wp schema diff

    wp schema migrate

    wp schema status

    wp schema rollback


==================================================
RELATIONS
==================================================

Relations are content concepts, not merely foreign keys.

Support eventually:

    belongs_to
    has_one
    has_many
    many_to_many
    polymorphic
    self-reference


Conceptual:

    class WP_Model_Relation {

        public function name();

        public function type();

        public function source();

        public function target();

        public function many();

        public function get($model);

        public function attach($model, $related);

        public function detach($model, $related);

        public function sync($model, $related);
    }


Example:

    'author' => [
        'type'  => 'relation',
        'model' => 'author',
    ]


==================================================
VALIDATION
==================================================

Schema should define validation rules.

Example:

    'email' => [
        'type' => 'email',
        'required' => true,
    ]

    'age' => [
        'type' => 'number',
        'min' => 18,
        'max' => 120,
    ]

    'slug' => [
        'type' => 'slug',
        'unique' => true,
    ]


Conceptual:

    class WP_Schema_Validator {

        public function validate($schema, $data);

        public function validate_field($field, $value);

        public function errors();

        public function valid();
    ]


Validation should happen before persistence.


==================================================
BLOCKS / COMPONENTS
==================================================

Long-term goal.

Support content structures such as:

    Page
      ↓
    content[]
      ├── Hero
      ├── Text
      ├── Gallery
      └── CTA


Conceptually:

    'content' => [
        'type' => 'blocks',

        'blocks' => [
            'hero' => [...],
            'text' => [...],
            'gallery' => [...],
        ],
    ]


This is inspired by Payload/Sanity-style structured content.


==================================================
DRAFTS / VERSIONS
==================================================

Long-term support:

    draft
    published
    versions
    revisions


Possible model usage:

    get_model('book', 123);

    get_model('book', 123, [
        'status' => 'draft',
    ]);

    get_model('book', 123, [
        'status' => 'published',
    ]);


Do not implement prematurely.


==================================================
WORDPRESS CORE RELATIONSHIP
==================================================

WP Schema should be WordPress-native.

Useful conceptual mappings:

    WP_Post_Type
        ↓
    WP_Model_Type

    WP_Post
        ↓
    WP_Model

    WP_Query
        ↓
    WP_Model_Query

    register_post_type()
        ↓
    register_model_type()


The project can copy useful architecture and conventions from WordPress core.

However:

    WP Schema != WP_Post

    WP Schema != WP_Query clone

The goal is to preserve the familiar developer experience while replacing
the assumptions around wp_posts/wp_postmeta storage.


==================================================
CORE COMPATIBILITY GOAL
==================================================

The architecture should make it possible for the project to eventually
be considered for WordPress core.

Do NOT assume a future core merge.

Instead optimize for:

    WordPress naming conventions
    WordPress coding conventions
    WordPress error handling
    WordPress hooks
    WordPress capability model
    WordPress APIs
    WordPress developer familiarity


Avoid unnecessary abstraction when a WordPress-native solution exists.


==================================================
DESIGN PRINCIPLES
==================================================

1. Schema first.

2. Schema describes content, not SQL.

3. Models represent content.

4. Repositories persist content.

5. Queries retrieve content.

6. Query builders handle SQL generation.

7. Database implementation must remain replaceable.

8. Validation belongs to the schema system.

9. Relations are first-class content concepts.

10. Migrations must be explicit and safe.

11. Never silently destroy user data.

12. Public API should remain small.

13. Internal architecture can be sophisticated.

14. Follow WordPress conventions where they make sense.

15. Do not blindly copy WordPress storage assumptions.

16. Avoid premature features.

17. Prefer extensibility through schema/field abstractions.

18. Keep SQL out of models.

19. Keep database details out of public schema definitions.

20. Treat the Schema AST as the long-term source of truth.


==================================================
INITIAL IMPLEMENTATION PRIORITY
==================================================

PHASE 1:

    WP_Schema
    WP_Schema_Field
    WP_Model_Type
    WP_Model
    WP_Model_Repository
    WP_Model_Query


PHASE 2:

    Schema Registry
    Validation
    Query Builder
    Database Schema
    Migrations


PHASE 3:

    Relations
    Object fields
    Arrays
    JSON
    Media


PHASE 4:

    Blocks
    Rich text
    Drafts
    Versions
    Localization


PHASE 5:

    REST
    GraphQL
    Type generation
    Admin UI


==================================================
WHEN MAKING ARCHITECTURAL DECISIONS
==================================================

Always ask:

    1. Is this a CONTENT concept or a DATABASE concept?

    2. Does this belong in Schema, Model, Repository, Query, or Database?

    3. Does this preserve the WordPress developer experience?

    4. Does this make future REST/GraphQL generation possible?

    5. Does this make future migrations safe?

    6. Does this unnecessarily couple the public API to MySQL?

    7. Does this move WP Schema closer to a schema-first content engine?

Prefer solutions that preserve the following architecture:

    Content Definition
          ↓
       WP Schema
          ↓
      Schema AST
          ↓
    ┌─────┼──────────┐
    ↓     ↓          ↓
  Model Query     Validation
    ↓     ↓
Repository
    ↓
Database


==================================================
CURRENT NORTH STAR
==================================================

WP Schema should feel like:

    WordPress
        +
    modern content modeling
        +
    typed schemas
        +
    relationships
        +
    safe persistence
        +
    generated APIs


Target conceptual experience:

    register_model_type(...)
            ↓
        Content Schema
            ↓
          Model
            ↓
        Query / Mutation
            ↓
          Database


The developer should not need to care how the schema is physically stored.

The schema should be the contract.
The model should be the content.
The repository should be the persistence boundary.
The query layer should be the retrieval boundary.
The database should be an implementation detail.
