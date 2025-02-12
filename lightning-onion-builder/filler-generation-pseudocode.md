# Filler generation pseudocode

The [bolt 4 specification](https://github.com/lightning/bolts/blob/master/04-onion-routing.md) contains a [go code example](https://github.com/lightning/bolts/blob/master/04-onion-routing.md#filler-generation) of the filler generation. In this document I'm trying to create a language independent (pseudocode) version of the filler generation that doesn't use fixed size packets.

## pseudocode

``` 
// Function to generate a filler byte array
FUNCTION generate_filler() 

    // Calculate the total filler size based on the number of hops and the maximum hop size
    filler_size = 1300 + max(hop_sizes) // Use the maximum hop size for the initial filler size

    // Initialize the filler array with the calculated size
    filler = init ARRAY of zeros with size filler_size

    // Loop trough all the hops.
    // The last hop does not obfuscate, so we skip it
    LOOP total_number_of_hops - 1 DO

        // Step 1: Get the current hop packet size 
        current_hop_payload_size = get_current_hop_payload_size(hops).

        // Step 2: Shift the filler array to the left by the current hop size
        if FIRST_ROUND then
            // For the first round, we don't need to shift the filler array
            // as it's already filled with zeros
            skip_left_shift
        else
            LOOP lenght_of_filler - current_hop_payload_size  DO
                // Shift each byte in the filler array to the left by current_hop_payload_size positions
                // [a,b,c,d,e] -> length of payload is 3 -> [d,e]
                filler[j] = filler[j + current_hop_payload_size]
            end LOOP
        end if

        // Step 3: Extend the filler with current hop payload size with 0 (zero's)
        // [1300] -> [1300 + current_payload_size]
        LOOP current_payload_size_length
            filler.add = 0 // extend the filler wiht payload size zeros

        // Step 4: Generate a pseudo-random byte stream using the current hop's rho key 
        stream_key = generate_key("rho", shared_secret_of_current_hop)
        stream_bytes = generate_cipher_stream(stream_key, 1300 + current_hop_size)

        // Step 5: Obfuscate the filler by XORing it with the generated stream bytes
        filler = filler XOR stream_bytes
 
    end LOOP

    // Step 6: Get the size of the last hop
    last_hop_size = hop_sizes[num_hops - 1]

    // Step 7: Cut the filler down to the correct length based on the total hop sizes
    total_hop_size = sum(hop_sizes) // Calculate the total size of all hops
    filler_length = total_hop_size - last_hop_size // Adjust for the last hop

    // Step 8: Return the final filler, adjusted to the correct length
    // to make sure the filler is 1300 bytes
    return filler[0 - filler_legnth]

end FUNCTION
```
