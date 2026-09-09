# Chapitre 1 -- Presentation de Builder Codes

Ce depot implemente un unique contrat Solidity, `src/BuilderCodes.sol`
(moins de 400 lignes, aucun autre fichier source dans `src/`) : un token
ERC-721 ou chaque NFT represente un code alphanumerique unique (par
exemple `"myapp"`) enregistre par un builder, avec une adresse de paiement
associee pour l attribution et la distribution de revenus. Le contrat
implemente explicitement l interface `ICodesRegistry` du standard
ERC-8021, un ERC dedie a l attribution de revenus par code -- le README
le mentionne directement en lien vers eip.tools.

Le contrat est upgradeable via le patron UUPS (Universal Upgradeable Proxy
Standard) : il herite de `Initializable`, `UUPSUpgradeable`, et definit un
`initialize()` a la place d un constructeur classique (le vrai
constructeur appelle `_disableInitializers()` pour empecher toute
initialisation directe de l implementation). Il herite egalement de
`ERC721Upgradeable` (OpenZeppelin), `AccessControlUpgradeable` (controle
d acces par roles), `Ownable2StepUpgradeable` (transfert de propriete en
deux etapes, plus sur qu un transfert direct), et `EIP712` (de Solady, pour
la verification de signatures typees). Le nom du token est "Builder
Codes", son symbole "BUILDERCODE".

La bibliotheque Solady (`LibBit`, `LibString`, `SignatureCheckerLib`) est
utilisee systematiquement pour les operations bas niveau -- une preference
pour des implementations optimisees en gas plutot que les equivalents
OpenZeppelin standard, un choix frequent dans les contrats Base recents
que cette bibliotheque a deja documente dans d autres parcours.
