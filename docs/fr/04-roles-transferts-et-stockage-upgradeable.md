# Chapitre 4 -- Roles, restriction des transferts et stockage upgradeable

Le contrat definit trois roles distincts via `AccessControlUpgradeable` :
`REGISTER_ROLE` (enregistrer des codes, directement ou par signature),
`TRANSFER_ROLE` (initier des transferts de tokens), et `METADATA_ROLE`
(mettre a jour l URI de base ou forcer un rafraichissement de metadonnees
via `updateMetadata`). La fonction `hasRole` est surchargee pour que le
proprietaire du contrat (`owner()`, issu de `Ownable2StepUpgradeable`)
possede implicitement tous les roles sans qu ils lui soient explicitement
attribues -- `super.hasRole(role, account) || account == owner()`.

La restriction des transferts merite une attention particuliere :
`transferFrom` est surchargee pour exiger `TRANSFER_ROLE` chez l appelant
via `_checkRole` avant de deleguer a l implementation standard
d OpenZeppelin. Le commentaire source precise explicitement que
`safeTransferFrom` d ERC721Upgradeable herite de cette meme verification
(aucune autre fonction ne peut initier de transfert), et que detenir
`TRANSFER_ROLE` ne suffit pas a deplacer un token unilateralement : il
faut toujours l approbation ERC-721 standard du proprietaire du token.
Cette architecture -- que le README nomme "registre central avec
registrars peripheriques" -- permet un deploiement controle de patrons de
transfert specifiques (marketplace controle, mecanisme de recuperation)
sans jamais retirer a un titulaire de code le controle final sur son
propre token.

Le stockage du contrat suit le patron ERC-7201 (stockage nomme,
resistant aux collisions lors des mises a niveau) : `RegistryStorage`
(qui contient le prefixe d URI et le mapping `tokenId => payoutAddress`)
est accede via un emplacement de stockage fixe et precalcule,
`REGISTRY_STORAGE_LOCATION`, dont le commentaire source documente
explicitement la formule de derivation --
`keccak256(abi.encode(uint256(keccak256("base.BuilderCodes")) - 1)) &
~bytes32(uint256(0xff))` -- et accede via un bloc `assembly` qui pointe le
slot de reference `$` directement sur cette constante. Seul le
proprietaire peut autoriser une mise a niveau (`_authorizeUpgrade`
surcharge avec `onlyOwner`), et `renounceOwnership` est explicitement
desactivee (elle revert systematiquement) pour eviter une perte
accidentelle de controle du contrat.
