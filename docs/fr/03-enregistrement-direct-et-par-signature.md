# Chapitre 3 -- Deux chemins d enregistrement : direct et par signature EIP-712

Le contrat expose deux fonctions externes pour enregistrer un nouveau
code, toutes deux aboutissant a la meme fonction interne `_register`, mais
avec des exigences d autorisation differentes.

`register(code, initialOwner, initialPayoutAddress)` est protegee par
`onlyRole(REGISTER_ROLE)` : seule une adresse explicitement autorisee
(un "registrar" peripherique, au sens de l architecture decrite au
chapitre 4) peut l appeler directement, en payant elle-meme le gas.

`registerWithSignature(code, initialOwner, initialPayoutAddress, deadline,
signer, signature)` permet un enregistrement gasless : n importe quelle
adresse peut soumettre la transaction, tant qu elle fournit une signature
EIP-712 valide produite hors-chaine par une adresse `signer` qui possede
`REGISTER_ROLE`. La fonction verifie d abord que `block.timestamp` n a pas
depasse `deadline` (sinon `AfterRegistrationDeadline`), verifie que
`signer` a bien le role requis via `_checkRole`, puis reconstruit le hash
structure EIP-712 attendu -- `keccak256(abi.encode(REGISTRATION_TYPEHASH,
keccak256(bytes(code)), initialOwner, initialPayoutAddress, deadline))`,
ou `REGISTRATION_TYPEHASH` encode le schema
`BuilderCodeRegistration(string code, address initialOwner, address
payoutAddress, uint48 deadline)` -- et verifie la signature via
`SignatureCheckerLib.isValidSignatureNow` (qui supporte a la fois les
signatures ECDSA classiques et les signatures de contrats ERC-1271, pour
des `signer` qui seraient eux-memes des smart contracts).

Ce patron delegue-la-payante-du-gas-a-un-tiers est courant pour les
registres publics : un builder peut obtenir une signature d un registrar
autorise sans que ce dernier n ait a soumettre lui-meme chaque
transaction, et sans que le builder n ait besoin d obtenir prealablement
un role quelconque sur le contrat.
