# Class delegation #
The `[`delegate`]` attribute is a special in-built attribute. A class delegation looks like this ...

```
TPie = class
  public
    procedure MethodA;                virtual;
    function MethodB: integer;      virtual;
    constructor Create;
  end;

TBlueberryPie = class( TPie)
  private
    [delegate] FBasePie: TPie;
  public
    function MethodB: integer;     override;
  public
    constructor Create( NumOfBerries: integer; BasePie: TPie);
  end;
```

The `[`delegate`]` attribute on a class data member (or readable class property) with a class type will have the effect of implying certain method declarations and implementations. The [delegate](delegate.md) attribute may only be used in a pascali solution.

If there are any explicitly abstract methods in the class (TPie in the above example) that are methods of the nearest common ancestor of the base class and the delegate class, there there is an implicit declaration and implementation as such...

```
procedure TBlueberryPie.MethodA;
begin
FBase.MethodA
end;
```

# Interface delegation #
The delgate attribute can also be applied to an interface data member (or readable property)

```
IIntf = interface
    procedure IntMeth1;
    procedure IntMeth2;
    ….
    procedure IntMeth10;
  end;

TBlueberryPie = class( TPie, IIntf)
  private
    [delegate] FBase: IIntf;
  private
    procedure IntMeth2;
  public
    constructor Create( NumOfBerries: integer; BasePie: TPie);
  end;
```

It is an error if the base class does not explicitly support the delegate interface pointer type. Any methods of the base interface pointer type that are not explicitly declared, or are abstract, are implicitly defined as being declared with private and delegating the function to the delegate data member or property.

For example, in the above, this is implied...

```
procedure TBlueberryPie.IntMeth1;
begin
FBase.IntMeth1
end;
```

# Union delegate #
The union-delegate attribute is a special inbuilt attribute which may only be used in pascali solutions. Applying to an IList`<T>`, where type T is an interface pointer type, it makes the following implicit declarations.

  * Declares another interface pointer type (name of first parameter, guild of second), which includes all the methods of the base interface, but with `'ForEach-'` prefixed on the names of the methods and properties.
  * Add methods Enlist(), Delist(), operator "+" and operator "-" to the union interface pointer type. These methods have signature as indicated in the following example.
  * Implicitly declares explicitly missing methods, and abstract methods of the union interface pointer, in the style of an observer pattern, as indicated in the following example.

This fragment of code in a solution ...

```
Type
IPie = interface
  _guid 1_
    procedure Method1;
    procedure Method2( Value: integer);
    function Method3: integer;
  end;

[union-delegate]
IPieUnion = interface
  _guid 2_
  end;

TPieUnion = class( TObject)
  private
    [union-delegate(IPieUnion] FPies: IList<IPie>;
    procedure ForEach-Method1;
    procedure ForEach-Method2( Value: integer);  virtual; abstract;
    procedure Delist( const Subracthend: IPie);
    procedure IPieUnion.ForEach-Method4 = AlternateNameMethod4;
  public
    constructor Create;
  end;

constructor TPieUnion.Create;
begin
FPies := TInterfaceList<IPie>.Create
end;

procedure TPieUnion.ForEach-Method1;
begin
SomeOverrideBehaviour()
end;


procedure TPieUnion.Delist( const Subracthend: IPie);
begin
FPies.Remove( Subtracthend);
DoSomethingElse()
end;
```

...would be equivalent to this fragment, without using union-delegate...

```
Type
IPie = interface
  _guid 1_
    procedure Method1;
    procedure Method2( Value: integer);
    function Method3: integer;
    procedure Method4;
  end;

IPieUnion = interface
  _guid 2_
    procedure ForEach-Method1;
    procedure ForEach-Method2;
    function ForEach-Method3: integer;
    procedure ForEach-Method4;
    procedure Enlist( const Addend: IPie);
    procedure Delist( const Subracthend: IPie);
    operator + ( const Addend: IPie);
    operator - ( const Subracthend: IPie);
  end;

TPieUnion = class( TObject, IPieUnion)
  private
    FPies: IList<IPie>;
    procedure ForEach-Method1;
    procedure ForEach-Method2( Value: integer);  virtual;
    function ForEach-Method3: integer;
    procedure AlternateNameMethod4;
    procedure IPieUnion.ForEach-Method4 = AlternateNameMethod4;
      procedure Enlist( const Addend: IPie);
    procedure Delist( const Subracthend: IPie);
    operator +( const Addend: IPie);
    operator - ( const Subracthend: IPie);
  public
    constructor Create;
  end;


constructor TPieUnion.Create;
begin
FPies := TInterfaceList<IPie>.Create
end;

procedure TPieUnion.ForEach-Method1;
begin
SomeOverrideBehaviour()
end;

procedure TPieUnion.ForEach-Method2( Value: integer);
var
  J: IPie;
begin
for J in FPies do
    J.Method2( Value)
end;

function TPieUnion.ForEach-Method3: integer;
var
  J: IPie;
begin
for J in FPies do
    result := J.Method2( Value)
end;

procedure AlternateNameMethod4;
var
  J: IPie;
begin
for J in FPies do
    J.Method4
end;

procedure TPieUnion.Enlist( const Addend: IPie);
begin
FPies.Add( Addend)
end;

procedure TPieUnion.Delist( const Subracthend: IPie);
begin
FPies.Remove( Subtracthend);
DoSomethingElse()
end;

operator  TPieUnion.+ ( const Addend: IPie);
begin
Enlist( Addend)
end;

operator TPieUnion.- ( const Subracthend: IPie);
begin
Delist( Addend)
end;
```