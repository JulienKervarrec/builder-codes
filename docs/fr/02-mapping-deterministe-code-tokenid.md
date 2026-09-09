# Chapitre 2 -- toTokenId et toCode : un mapping deterministe par bit-packing

Le coeur technique de ce contrat est la conversion bidirectionnelle et
deterministe entre une chaine de caracteres (le code, jusqu a 32
caracteres) et un entier `uint256` (le token ID), sans jamais stocker
cette correspondance dans un mapping on-chain -- elle est recalculee a
chaque fois par pure arithmetique de bits.

`isValidCode` verifie d abord qu un code fait entre 1 et 32 caracteres,
puis que chaque caractere appartient a l alphabet autorise :
`ALLOWED_CHARACTERS = "0123456789abcdefghijklmnopqrstuvwxyz_"` (chiffres,
minuscules, underscore -- 38 caracteres). Cette verification n itere pas
caractere par caractere en Solidity classique : elle delegue a
`LibString.is7BitASCII(code, ALLOWED_CHARACTERS_LOOKUP)`, ou
`ALLOWED_CHARACTERS_LOOKUP` est une constante `uint128` precalculee (une
table de correspondance en bits, un octet du commentaire source precise
qu elle vaut `LibString.to7BitASCIIAllowedLookup(ALLOWED_CHARACTERS)`) --
un test de validite en temps constant plutot qu une boucle.

`toTokenId` caste la chaine de caracteres en `bytes32` (ce qui aligne les
octets du code a gauche, "high-endian", avec des zeros de bourrage a
droite), puis decale ce `bytes32` vers la droite d un nombre d octets egal
a `32 - bytes(code).length` -- annulant le decalage gauche implicite du
cast bytes->bytes32, pour obtenir un entier ou seuls les octets
significatifs du code sont non nuls, aligne a droite comme un entier
normal. `toCode` fait l inverse : elle compte les zeros de poids fort du
token ID via `LibBit.clz` (count leading zeros), les convertit en nombre
d octets, redecale l entier vers la gauche pour reconstituer un
`bytes32` aligne comme une "petite chaine" Solady, verifie via
`LibString.normalizeSmallString` que le resultat est bien une forme
canonique (sinon `InvalidTokenId`), puis extrait la chaine via
`LibString.fromSmallString` et revalide sa conformite via `isValidCode`.

Cette double verification (a l aller et au retour) garantit une bijection
stricte : un token ID donne ne peut correspondre qu a au plus un seul
code valide, et vice-versa, sans jamais necessiter de mapping
`code => tokenId` stocke en storage -- une economie de gas significative
pour un registre potentiellement utilise par des millions de builders.
