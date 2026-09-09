# Chapitre 5 -- Limites et perimetre de ce parcours

Ce parcours couvre la presentation generale du contrat unique de ce
depot et sa conformite a l interface ERC-8021, le mecanisme de conversion
bidirectionnelle et deterministe entre code et token ID par bit-packing
(`toTokenId`/`toCode`), les deux chemins d enregistrement (direct et par
signature EIP-712 gasless), et l architecture de roles, de restriction
des transferts et de stockage upgradeable ERC-7201.

Sont volontairement laisses hors champ : le dossier `test/` (la suite de
tests Foundry qui accompagne le contrat) ; le dossier `script/`
(scripts de deploiement) ; le fichier `.gitmodules` et les dependances
externes en detail (OpenZeppelin upgradeable, Solady) au-dela de leur
usage cite aux chapitres precedents ; le detail complet de l implementation
ERC-721 standard heritee d OpenZeppelin (approve, getApproved,
setApprovalForAll, et le reste de l interface non surcharge par ce
contrat) ; et l outillage Foundry (Makefile, foundry.toml,
foundry.lock) utilise pour compiler et tester le contrat.

L objectif reste le meme que pour les parcours precedents de cette
bibliotheque : comprendre precisement comment ce contrat, volontairement
compact (un seul fichier source), implemente un registre de codes
verifiable, upgradeable et econome en gas, sans pretendre couvrir
l integralite de son dispositif de tests ou de deploiement.
