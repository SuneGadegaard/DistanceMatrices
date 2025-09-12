# DistanceMatrices
Helper class for generating distances matrices for road networks based on open route service.

# Example
```python
from DistanceMatrixMaker import DistanceMatrix
import pandas as pd
# Create an instance of the DistanceMatrix class
dstMtrx = DistanceMatrix()

# Add three addresses one at a time
dstMtrx.addAddress("Nørregade 17, 8783 Hornsyld")
dstMtrx.addAddress("Bjerrevej 83, 8700 Horsens")
dstMtrx.addAddress("Fulgesangs Allé 4, 8210 Aarhus V")

# Add two more addresses in one-go
dstMtrx.addAddresses(["Gimlevej 1, 8660 Skanderborg", "Nordmarksvej 9, 7190 billund"])

# Print addresses
print(dstMtrx.Addresses)

# Set the API key for Open Route Service
dstMtrx.setAPIKey("YOUR_API_KEY_GOES_HERE")

# Create the distance matrix
dstMtrx.makeMatrix()

# Print the distance matrix in km and the duration matrix in minutes
df_dist_km = pd.DataFrame(dstMtrx.dist_km, index=dstMtrx.Addresses, columns=dstMtrx.Addresses)
df_dur_min = pd.DataFrame(dstMtrx.dur_min, index=dstMtrx.Addresses, columns=dstMtrx.Addresses)
print(df_dist_km.to_string())
print(df_dur_min.to_string())
```
