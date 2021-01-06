# SU2 CNumerics

### Logic in `CNumerics.h`

1. `Class CSourceBase_TurbSA : public CNumerics`
2. `class CSourcePieceWise_TurbSA final : public CSourceBase_TurbSA`

3. `class CSourcePieceWise_TurbSA_COMP final : public CSourceBase_TurbSA`
4. `class CSourcePieceWise_TurbSA_E final : public CSourceBase_TurbSA`
5. `class CSourcePieceWise_TurbSA_E_COMP : public CSourceBase_TurbSA `
6. `class CSourcePieceWise_TurbSA_Neg : public CSourceBase_TurbSA`

7. `class CSourcePieceWise_TurbSST final : public CNumerics`

Above shows there are several different **subclasses** all inherited from the **base class** `CSourceBase_TurbSA` which is inherited from `CNumerics`. Again, subclasses have all contained in the base class. 