		const GRADIENT_CALC_CONFIG = {
		    directCalcWindowSize: 2,   // Number of points for the initial "direct" gradient calc (uses the segment between targetIndex and targetIndex-1)
		    maxWindowSize: 20,          // Max number of points in the fallback weighted window (this means up to maxWindowSize-1 segments)
		    validGradientMin: -25,     // Min valid gradient percentage for direct use
		    validGradientMax: 25       // Max valid gradient percentage for direct use
		};	

          // Elevation services configuration
        const ELEVATION_SERVICES = [
            {
                name: 'OpenTopoData',
                processor: async (coords) => {
                    const geoJson = {
                        type: "LineString",
                        coordinates: coords
                    };
                    const response = await fetch('https://api.opentopodata.org/v1/srtm30m?locations=' + 
                        encodeURIComponent(JSON.stringify(geoJson)), {
                        signal: AbortSignal.timeout(3000)
                    });
                    const data = await response.json();
                    return data.results?.map(r => r.elevation) || [];
                },
                batchSize: 100
            },
            {
                name: 'Open-Elevation',
                processor: async (coords) => {
                    const points = coords.map(coord => `${coord[1]},${coord[0]}`).join('|');
                    const response = await fetch(`https://api.open-elevation.com/api/v1/lookup?locations=${points}`, {
                        signal: AbortSignal.timeout(3000)
                    });
                    const data = await response.json();
                    return data.results?.map(r => r.elevation) || [];
                },
                batchSize: 50
            }
        ];
        
