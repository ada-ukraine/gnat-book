# **Appendix B.** **Glossary**

**Access type**. An access type has values that designate aliased objects. Access types correspond to "pointer types" or "reference types" in some other languages.

`Aliased`. An aliased view of an object is one that can be designated by an access value. Objects allocated by allocators are aliased. Objects can also be explicitly declared as aliased with the reserved word aliased. The Access attribute can be used to create an access value designating an aliased object.

`Class`. A class is a set of types that is closed under derivation, which means that if a given type is in the class, then all types derived from that type are also in the class. The set of types of a class share common properties, such as their primitive operations.

**Compilation unit**. The text of a program can be submitted to the compiler in one or more compilations. Each compilation is a succession of compilation units. A compilation unit contains either the declaration, the body, or a renaming of a program unit.

**Compile-time rules**. Rules that are enforced at compile type.

**Complete context**. Language construct over which overload resolution must be performed without examining a larger portion of the program. Each of the following constructs is a complete context: a context item, a declarative item or declaration, a statement, a pragma argument association, and the expression of a case statement [AAR95, Section 8.6(4-9)].

**Composite type**. A composite type has components.

`Construct`. A construct is a piece of text (explicit or implicit) that is an instance of a syntactic category defined under "Syntax."

**Controlled type**. A controlled type supports user-defined assignment and finalization. Objects are always finalized before being destroyed.

`Declaration`. A declaration is a language construct that associates a name with (a view of) an entity. A declaration may appear explicitly in the program text (an explicit declaration), or may be supposed to occur at a given place in the text as a consequence of the semantics of another construct (an implicit declaration).

`Definition` (view) All declarations contain a definition for a view of an entity. A view consists of an identification of the entity (the entity of the view), plus view-specific characteristics that affect the use of the entity through that view (such as mode of access to an object, formal parameter names and defaults for a subprogram, or visibility to components of a type). In most cases, a declaration also contains the definition for the entity itself (a renaming declaration is an example of a declaration that does not define a new entity, but instead defines a view of an existing entity (see ARM 8.5)).

**Derived type**. A derived type is a type defined in terms of another type, which is the parent type of the derived type. Each class containing the parent type also contains the derived type. The derived type inherits properties such as components and primitive operations from the parent. A type together with the types derived from it (directly or indirectly) form a derivation class.

**Discrete type**. A discrete type is either an integer type or an enumeration type. Discrete types may be used, for example, in case statements and as array indices.

`Discriminant`. A discriminant is a parameter of a composite type. It can control, for example, the bounds of a component of the type if that type is an array type. A discriminant of a task type can be used to pass data to a task of the type upon creation.

**Dynamic semantics** (see run-time semantics).

**Elementary type**. An elementary type does not have components.

**Enumeration type**. An enumeration type is defined by an enumeration of its values, which may be named by identifiers or character literals.

`Exception`. An exception represents a kind of exceptional situation; an occurrence of such a situation (at run time) is called an exception occurrence. To raise an exception is to abandon normal program execution so as to draw attention to the fact that the corresponding situation has arisen. Performing some actions in response to the arising of an exception is called handling the exception.

`Execution`. The process by which a construct achieves its run-time effect is called execution. Execution of a declaration is also called elaboration. Execution of an expression is also called evaluation.

`Expanded_Name`. A selected component is called an expanded name if according to the visibility rules, at least one possible interpretation of its prefix denotes a package or an enclosing named construct (directly, not through a subprogram renaming declaration or generic renaming declaration) [AAR95, Section 4.1.3(4)].

**Generic unit**. A generic unit is a template for a (nongeneric) program unit; the template can be parameterized by objects, types, subprograms, and packages. An instance of a generic unit is created by a generic instantiation. The rules of the language are enforced when a generic unit is compiled, using a generic contract model; additional checks are performed upon instantiation to verify the contract is met. That is, the declaration of a generic unit represents a contract between the body of the generic and instances of the generic. Generic units can be used to perform the role that macros sometimes play in other languages.

**Integer type**. Integer types comprise the signed integer types and the modular types. A signed integer type has a base range that includes both positive and negative numbers, and has operations that may raise an exception when the result is outside the base range. A modular type has a base range whose lower bound is zero, and has operations with "wraparound" semantics. Modular types subsume what are called "unsigned types" in some other languages.

**Legality rules** (see Compile-time rules).

**Library unit**. A library unit is a separately compiled program unit, and is always a package, subprogram, or generic unit. Library units may have other (logically nested) library units as children, and may have other program units physically nested within them. A root library unit, together with its children and grandchildren and so on, form a subsystem.

**Limited type**. A limited type is (a view of) a type for which the assignment operation is not allowed. A nonlimited type is a (view of a) type for which the assignment operation is allowed.

**Name resolution**. Process that establishes a mapping between names and the defining entity referred to by the names at each point in the program. In the context of the GNAT semantic analyzer, name resolution involves to link each node that denotes an entity with its corresponding defining-entity node.

**Name resolution rules.** Compile-time rules used in name resolution, including overload resolution.

`Object`. An object is either a constant or a variable. An object contains a value. An object is created by an object declaration or by an allocator. A formal parameter is (a view of) an object. A subcomponent of an object is an object.

`Package`. Packages are program units that allow the specification of groups of logically related entities. Typically, a package contains the declaration of a type (often a private type or private extension) along with the declarations of primitive subprograms of the type, which can be called from outside the package, while their inner workings remain hidden from outside users.

`Pragma`. A pragma is a compiler directive. There are language-defined pragmas that give instructions for optimization, listing control, etc. An implementation may support additional (implementation-defined) pragmas.

**Primitive operations**. The primitive operations of a type are the operations (such as subprograms) declared together with the type declaration. They are inherited by other types in the same class of types. For a tagged type, the primitive subprograms are dispatching subprograms, providing run-time polymorphism. A dispatching subprogram may be called with statically tagged operands, in which case the subprogram body invoked is determined at compile time. Alternatively, a dispatching subprogram may be called using a dispatching call, in which case the subprogram body invoked is determined at run time.

**Private extension**. A private extension is like a record extension, except that the components of the extension part are hidden from its clients.

**Private type**. A private type is a partial view of a type whose full view is hidden from its clients.

**Program unit**. A program unit is either a package, a task unit, a protected unit, a protected entry, a generic unit, or an explicitly declared subprogram other than an enumeration literal. Certain kinds of program units can be separately compiled. Alternatively, they can appear physically nested within other program units.

**Protected type**. A protected type is a composite type whose components are protected from concurrent access by multiple tasks.

**Record extension**. A record extension is a type that extends another type by adding additional components.

**Record type**. A record type is a composite type consisting of zero or more named components, possibly of different types.

**Run-time semantics** (dynamic semantics). Definition of the run-time effect of each construct.

**Scalar type**. A scalar type is either a discrete type or a real type.

`Subtype`. A subtype is a type together with a constraint, which constrains the values of the subtype to satisfy a certain condition. The values of a subtype are a subset of the values of its type.

**Tagged type**. The objects of a tagged type have a run-time type tag, which indicates the specific type with which the object was originally created. An operand of a class-wide tagged type can be used in a dispatching call; the tag indicates which subprogram body to invoke. Nondispatching calls, in which the subprogram body to invoke is determined at compile time, are also allowed. Tagged types may be extended with additional components.

**Task type**. A task type is a composite type whose values are tasks, which are active entities that may execute concurrently with other tasks. The top-level task of a partition is called the environment task.

`Type`. Each object has a type. A type has an associated set of values, and a set of primitive operations which implement the fundamental aspects of its semantics. Types are grouped into classes. The types of a given class share a set of primitive operations. Classes are closed under derivation; that is, if a type is in a class, then all of its derivatives are in that class.

`View`. (See Definition.)
