import openrouteservice as ors
from openrouteservice import distance_matrix, geocode

class DistanceMatrix:
    class DistanceMatrix:
        """
        A helper class for building distance and duration matrices using the
        OpenRouteService (ORS) API.

        This class allows you to:
          - Store and manage a list of addresses.
          - Geocode addresses into coordinates.
          - Compute pairwise travel distances and durations using the ORS Matrix API.
          - Work with both raw values (meters/seconds) and converted units (km/min).

        Attributes
        ----------
        Addresses : list of str
            List of addresses to be geocoded and included in the matrix.
        API_KEY : str
            ORS API key for authentication. Defaults to an empty string.
        profile : str
            Routing profile to use for queries. Defaults to "driving-car".
            Examples: "driving-car", "driving-hgv", "cycling-regular",
            "cycling-road", "cycling-mountain", "cycling-electric", "cycling-safe",
            "foot-walking", "foot-hiking", "wheelchair".
        client : openrouteservice.Client or None
            ORS client instance, created after setting the API key.
        dist_m : list of lists of float or None
            Distance matrix in meters.
        dur_sec : list of lists of float or None
            Duration matrix in seconds.
        dist_km : list of lists of float or None
            Distance matrix in kilometers.
        dur_min : list of lists of float or None
            Duration matrix in minutes.

        Methods
        -------
        addAddress(address)
            Add a single address to the list.
        addAddresses(addresses)
            Add multiple addresses at once.
        setAPIKey(API_KEY)
            Store the ORS API key for authentication.
        setProfile(profile)
            Set the routing profile (e.g., "driving-car", "cycling-regular").
        makeMatrix()
            Build distance and duration matrices for the stored addresses.
    """
    def __init__(self):
        """ Initialize the class with default attributes for working with
            OpenRouteService distance and duration matrices.

            Attributes
            ----------
            Addresses : list
                A list of addresses (strings) to be geocoded and used in routing/matrix queries.
            API_KEY : str
                The API key for authenticating with the OpenRouteService API.
                Defaults to an empty string. Can be set using setAPIKey()
            profile : str
                The routing profile to use for queries. Defaults to "driving-car".
                Valid profiles include "driving-car", "driving-hgv", "cycling-regular",
                "cycling-road", "cycling-mountain", "cycling-electric", "cycling-safe",
                "foot-walking", "foot-hiking", and "wheelchair". Can be set using setProfile()
            client : openrouteservice.Client or None
                The ORS client instance. Initialized to None until set with a valid API key.
            dist_m : list or None
                Pairwise distances in meters, as returned by the ORS Matrix API.
            dur_sec : list or None
                Pairwise travel times in seconds, as returned by the ORS Matrix API.
            dist_km : list or None
                Pairwise distances in kilometers (converted from `dist_m`).
            dur_min : list or None
                Pairwise travel times in minutes (converted from `dur_sec`).
        """
        self.Addresses = []
        self.API_KEY = ""
        self.profile = "driving-car"
        self.client = None
        self.dist_m = None
        self.dur_sec = None
        self.dist_km = None
        self.dur_min = None

    def addAddress(self, address: str):
        """ Add a new address to the list of stored addresses.

            Parameters
            ----------
            address : str
                The address string to be added. This will later be used for
                geocoding and distance/duration matrix calculations.

            Returns
            -------
            None
        """
        self.Addresses.append(address)

    def addAddresses(self, addresses: list):
        """ Add multiple addresses to the list of stored addresses.

            Parameters
            ----------
            addresses : list of str
                A list of address strings to be added. Each element must be a string.

            Returns
            -------
            None

            Raises
            ------
            ValueError
                If the input is not a list, or if any element in the list is not a string.

            Notes
            -----
            - This function validates both the type of the input (`list`) and the type
              of each element (`str`).
            - Addresses are appended in the order they appear in the input list.
            """
        if not isinstance(addresses, list):
            raise ValueError("The addAddresses function only accepts a list of strings")

        if not all(isinstance(a, str) for a in addresses):
            raise ValueError("All elements in the list must be strings")

        self.Addresses.extend(addresses)

    def setAPIKey(self, API_KEY: str):
        """ Set the API key used for authenticating with the OpenRouteService API.

            Parameters
            ----------
            API_KEY : str
                The API key string obtained from the OpenRouteService developer portal.

            Returns
            -------
            None

            Notes
            -----
            - This method only stores the API key in the instance attribute `API_KEY`.
            - To obtain an API visit https://openrouteservice.org/dev/#/home
        """
        self.API_KEY = API_KEY

    def makeMatrix(self):
        """ Build distance and duration matrices for the stored addresses using the
            OpenRouteService (ORS) Matrix API.

            This method:
              1. Initializes an ORS client with the stored API key.
              2. Geocodes all addresses in `self.Addresses` to coordinates.
              3. Requests a matrix of pairwise distances and durations.
              4. Stores the results in both raw (meters/seconds) and converted
                 (kilometers/minutes) formats.

            Returns
            -------
            None

            Raises
            ------
            openrouteservice.exceptions.ApiError
                If the ORS Matrix API request fails (e.g., invalid API key, quota exceeded).
            ValueError
                If any address cannot be geocoded.

            Attributes Set
            --------------
            client : openrouteservice.Client
                ORS client initialized with the current API key.
            dist_m : list of lists of float
                Distance matrix in meters.
            dur_sec : list of lists of float
                Duration matrix in seconds.
            dist_km : list of lists of float
                Distance matrix in kilometers.
            dur_min : list of lists of float
                Duration matrix in minutes.

            Notes
            -----
            - The routing profile is determined by `self.profile` (default: "driving-car").
            - Distances and durations are not symmetric due to the profile chosen and the road network
              (e.g., one-way streets).
            """
        self.client = ors.Client(key=self.API_KEY)
        coords = [self.__geocode_address(a) for a in self.Addresses]

        # Request matrix (driving-car). metrics=['distance','duration'] ---
        matrix = distance_matrix.distance_matrix(
            self.client,
            locations=coords,
            profile=self.profile,
            metrics=["distance", "duration"],  # meters & seconds
            units="m"
        )
        #  Build DataFrames (meters and seconds)
        self.dist_m = matrix.get("distances")
        self.dur_sec = matrix.get("durations")
        self.dist_km = [[x / 1000.0 for x in row] for row in self.dist_m]
        self.dur_min = [[x / 60.0 for x in row] for row in self.dur_sec]

    # Geocode each address (returns Pelias results) ---
    def __geocode_address(self, addr, country="DK"):
        """ Geocode a textual address into geographic coordinates using the ORS Pelias geocoder.

            Parameters
            ----------
            addr : str
                The full address string to geocode.
            country : str, optional
                ISO country code to restrict the search to. Defaults to "DK" (Denmark).

            Returns
            -------
            list of float
                A two-element list `[lon, lat]` with the longitude and latitude of the address.

            Raises
            ------
            ValueError
                If the address cannot be geocoded (no features returned by the API).

            Notes
            -----
            - This method uses the OpenRouteService Pelias geocoder via
              `openrouteservice.geocode.pelias_search`.
            - Only the first match (`size=1`) is returned.
            - The coordinate order follows GeoJSON convention: `[longitude, latitude]`.
        """
        res = geocode.pelias_search(self.client, text=addr, country=country, size=1)
        feats = res.get("features", [])
        if not feats:
            raise ValueError(f"Could not geocode: {addr}")
        # geometry.coordinates is [lon, lat]
        return feats[0]["geometry"]["coordinates"]

    def setProfile(self, profile: str="driving-car"):
        """ This function sets the profile-attribute for the distance matrix
            It controls how its assumed we move from a to b
            The default option is 'driving-car'

            Parameters:
                ----------
                profile : {"driving-car", "driving-hgv", "cycling-regular", "cycling-safe", "cycling-electric"}
                    Provides the profile used when calculating distances and durations

            Notes:
                ----------
                Other options for the profile is probably available. Above, some of the more popular are listed
                Visit https://giscience.github.io/openrouteservice/api-reference/endpoints/directions/routing-options
                for more options for the profile, or simply google your way to it.
        """
        self.profile =profile


if __name__ == "__main__":
    # Create an instance of the DistanceMatrix class
    dstMtrx = DistanceMatrix()

    # Add three addresses one at a time
    dstMtrx.addAddress("Nørregade 17, 8783 Hornsyld")
    dstMtrx.addAddress("Bjerrevej 86, 8700 Horsens")
    dstMtrx.addAddress("Fulgesangs Allé 4, 8210 Aarhus V")

    # Add two more addresses in one go
    dstMtrx.addAddresses(["Gimlevej 1, 8660 Skanderborg", "Nordmarksvej 9, 7190 billund"])

    # Print addresses - just to inspect
    print(dstMtrx.Addresses)

    # Set the API key for Open Route Service
    dstMtrx.setAPIKey("This_Is_Where_Your_ORS_API_Key_Goes")

    # Create the distance matrix
    dstMtrx.makeMatrix()

    # Print the distance matrix in km and the duration matrix in minutes
    import pandas as pd
    df_dist_km = pd.DataFrame(dstMtrx.dist_km, index=dstMtrx.Addresses, columns=dstMtrx.Addresses)
    df_dur_min = pd.DataFrame(dstMtrx.dur_min, index=dstMtrx.Addresses, columns=dstMtrx.Addresses)
    print("Distances in kilometres")
    print(df_dist_km.to_string())
    print("Durations in minutes")
    print(df_dur_min.to_string())
