Hva kan verden lære av Haskell?
===============================

Tvungen immutability
--------------------

- `final` er default
- Sett gang på gang at det burde vært tilfellet i alle språk
- Men vi er jo ikke et elfenbenstårn heller...
- Du har et par opt-outs i Haskell (er jo mulig å gjøre IO)
	+ unsafePerformIO
	+ IORef / MVar m/venner
	+ FFI
- Trenger sjelden en opt-out
- Atomisk tilstandsbytte fordi funksjonskall (eksempel)

```haskell
gcd :: Integral a => a -> a -> a
gcd a 0 = a
gcd 0 b = gcd b 0
gcd a b = gcd b (a `rem` b)
```

```c
int gcd(int a, int b) {
	while (b) {
		int temp_b = a % b;
		a = b;
		b = temp_b;
    }
    return a;
}
```

```python
def gcd(a, b):
    while b:
         a, b = b, a % b
    return a
```

```java
import com.fasterxml.jackson.databind.JsonCreator;
// ...

public class Bruker {
	private final String brukerNavn;
	private final int brukerId;

    @JsonCreator
	public Bruker(@JsonProperty("brukerNavn") String brukerNavn, @JsonProperty("brukerId") int brukerId) {
		this.brukerNavn = brukerNavn;
		this.brukerId = brukerId;
	}

    public String brukerNavn() { return brukerNavn; }
    public int brukerId() { return brukerId; }

    @Override
	public String toString() {
		return "Bruker{brukerNavn='" + brukerNavn + "',brukerId=" + brukerId + "}";
	}

    @Override
	public boolean equals(Object other) {
		if (!getClass().equals(other.getClass())) return false;
		Bruker bruker = (Bruker) other;
		return bruker.brukerId == brukerId && (brukerNavn == null && bruker.brukerNavn == null || brukerNavn.equals(brukerNavn));
	}

	@Override
	public int hashCode() {
		return brukerNavn == null ? brukerId : brukerId + brukerNavn.hashCode();
	}

	public static void main(String args[]) {
		ObjectMapper mapper = new ObjectMapper();
		System.out.println(mapper.writeValueAsString(new Bruker("Robin", 1337)));
	}
}
```

```haskell
{-# LANGUAGE DeriveDataTypeable #-}
import Data.Data
import Data.Typeable()
import Data.Aeson.Generic

data Bruker = Bruker { brukerNavn :: String, brukerId :: Int } deriving (Show, Eq, Data, Typeable)

main :: IO ()
main = do putStrLn $ encode $ Bruker "Robin" 1337
```

```python
import json
import collections

Bruker = collections.namedtuple('Bruker', ['brukerNavn', 'brukerId'])
robin = Bruker('Robin', 1337)
if __name__ == '__main__':
    print(json.dumps(robin.__dict__))
```

HM typeinferens
---------------

- Hindley-Milner type-inferens
- Skal ha fordelene av et sterk typesystem
- Uten at det går i veien for deg
- Mer å si her
- Interferens funker nesten alltid (eksempel på noe som ikke virker)

Isolasjon av tilstandsrik kode
------------------------------

- IO-monaden 'smitter'
- Er lett å se hvor grensene for kode som kan 'gjøre' ting går
- Språket hjelper deg å implementere fornuftig kodestandard (holde sideeffekter adskilt fra logikk)
- Mønsteret kan implementeres i andre språk også (eksempel?)
- Eksempel på imparativ Haskell-kode

Typeklasser som polymorfisme
----------------------------

- Typeklasser er den mest fornuftige måten jeg har sett å få vtables på
- Sammenlikning med interfaces + implements, hvorfor forskjellig?
- Fordeler med typeclass dispatch vs tradisjonell dispatch, eksempel
- "Åpen" verden, du kan utvide mine typer
- Eksempler: Eq, Ord
- Tegning / forklaring av hvordan koden kompileres

Arv
---

- Egentlig samme punkt som siste
- Kan begrense typer (*må* være en sammenliknbar for å kunne implementere sorterbar)
- Egentlig bare interface inheritance
- Jeg liker det sånn

Hurtig eksperimentering
-----------------------

- Flere meninger
- ghci er kjempeflott
- -XLanguageFeature er kjempeflott
- Du har opt-in for eksperimentelle features hele tiden
- Eksempel: -XGADT
- Avoid (success at all costs) vs avoid success (at all costs) (spj, smarlow)

Parallelisering og concurrency
------------------------------

- forkIO
- STM
- Control.Parallel(par, pseq)
- Veldig enkelt å legge på i ettertid
- God ROI, skyldes i stor grad immutability
- GHC-runtime 

Veldig hjelpsom community
-------------------------

- Ikke så mye å si
- #Haskell
- Planet Haskell
- StackOverflow-gjengen
- Mailing lister (ghc)

Hva kan Haskell lære av Haskell?
--------------------------------

- Enkelte ting burde brekkes for bakoverkompatibilitet
- map / fmap (map burde vært Functor)
- length / genericLength (length burde vært på en typeklasse)
- Strenger burde ikke være lister av Char
	+ type String = [Char]
	+ Stringe er et alias, ikke en ekte type
	+ Kan blant annet ikke melde seg på typeklasser...
- Namespace clashing (spesielt record-scoping)
- Monomorfisme (eksempel)
- Burde ikke hatt så mange partial functions i stdlib (personlig mening)

Hva kan Haskell lære av verden?
===============================

Nyttig informasjon av feil (aka Stacktraces)
--------------------------------------------

- *** Exception: Prelude.head: empty list
- Ok, hva nå?
- Det sies at dette ikke er et alvorlig savn
- Jeg er ikke enig
- Mulig det er fordi jeg ikke kan kode
- Folk pleier jo å si at de ikke har bugs når det kompilerer

Slutte å lage monad tutorials
-----------------------------

- Kulturproblem?
- Det er ikke essensielt å forstå konseptet monads
- Det lærer man etter hvert som man bruker det anyway
- Føles som en glorification av ting som er vanskelig
- Man burde fokusere på å gjøre ting lett
- Spesifikt er lettere enn generelt

Kultur for å gjøre "vanskelige" ting
------------------------------------

- monads
- arrow (HXT eksempel)
- lens (eksempel)
- pipes
- template haskell (yesod)

Mye sjargong i community
-----------------------

- Veldig mange folk med mattematisk bakgrunn
- Noen kan uttrykke seg litt vanskelig

Trenger å standardisere på strengtyper
--------------------------------------

- type String = [Char]
- ByteString
- ByteString.Lazy
- Text
- Text.Lazy

Mye fokus på generelle og abstrakte biblioteker
-----------------------------------------------

- Veldig generelle (og sikkert nyttige) konsepter
- Men vanskelig å forstå
- Virker å ha høyere fokus og status enn praktiske implementasjoner
- Mange unntak også

Dokumentasjon
-------------

- Det viktigste for alle som ønsker seg suksess
- Dok er generelt dårligere enn i andre språk (ja, det finnes unntak)
- Korte leketøyseksempler eller ingen eksempler
- Utdatert
- Stygg
- Mye rart (og varierer jo også)

Testing
-------

Verktøystøtte
-------------

- Med et sterk typesystem burde det være lettere å lage gode verktøy
- Enkelte er bra, eg hlint er fantastisk
- "feilede" / treige prosjekter:
	+ leksah
	+ yi
- Ønsker meg en debugger
- Ja, jeg gjør feil selv med dette typesystemet

Pakking / installasjon
----------------------

- cabal ligger noen år etter andre plattformer
- cabal har et eget språk
- ikke sikker på om jeg liker at pakker innstalleres til ghc-installasjonen:
	+ gjør det dyrt å gjøre ting som virtuelle installasjoner
	+ gjør at det er lett å ødelegge installasjonen
	+ fiks ting med svenskeknappen
- 
