
## SDR p-adic processing.

1. Add detailed output showing the p-adic representations
2. Show the conversion process from real numbers to p-adic numbers
3. Display statistics about the noise filtering
4. Print both original and filtered signal samples

The code now shows:
- The initial parameters (number of samples, p-adic base, precision)
- Original signal samples
- P-adic representations
- Results of noise filtering
- Signal statistics including p-adic norms

Key additions:
- `print_padic()` function to display p-adic numbers
- `print_signal_samples()` to show multiple representations
- Statistics calculation for p-adic norms
- Reduced sample size to 100 for clearer output

when you run the code, you should see the complete processing pipeline with actual values.

Initializing SDR simulation with:
Number of samples: 100
P-adic base (p): 7
Precision: 5

Generating sample SDR data (AM signal with noise)...

Original signal samples (first 5):
Sample 0: 0.034019
Sample 1: 0.151606
Sample 2: 0.395259
Sample 3: 0.573022
Sample 4: 0.664502

Converting to p-adic representation...

Signal samples (showing 5 samples starting from 0):
Sample Index | P-adic Representation | Real Value | P-adic Norm
------------------------------------------------------------
          0 | (4, 4, 4, 1, 0)_7 |   571.0000 |     1.0000
          1 | (0, 0, 3, 0, 1)_7 |  2548.0000 |     0.0204
          2 | (0, 4, 2, 5, 2)_7 |  6643.0000 |     0.1429
          3 | (5, 3, 0, 0, 4)_7 |  9630.0000 |     1.0000
          4 | (3, 6, 3, 4, 4)_7 | 11168.0000 |     1.0000

Applying p-adic noise filtering with threshold 0.100000
Filtered 4 samples out of 100

After noise filtering:

Signal samples (showing 5 samples starting from 0):
Sample Index | P-adic Representation | Real Value | P-adic Norm
------------------------------------------------------------
          0 | (4, 4, 4, 1, 0)_7 |   571.0000 |     1.0000
          1 | (0, 0, 0, 0, 0)_7 |     0.0000 |     0.0000
          2 | (0, 4, 2, 5, 2)_7 |  6643.0000 |     0.1429
          3 | (5, 3, 0, 0, 4)_7 |  9630.0000 |     1.0000
          4 | (3, 6, 3, 4, 4)_7 | 11168.0000 |     1.0000

Signal Statistics:
Maximum p-adic norm: 1.000000
Average p-adic norm: 0.840000


=== Code Execution Successful ===


```
#include <stdio.h>
#include <stdlib.h>
#include <complex.h>
#include <math.h>

// Structure definitions remain the same
typedef struct {
    long *digits;
    int length;
    int p;
} PAdicNumber;

typedef struct {
    PAdicNumber** samples;
    int num_samples;
    int p;
} PAdicSignal;

// Initialize a p-adic number
PAdicNumber* init_padic(int p, int length) {
    PAdicNumber* num = (PAdicNumber*)malloc(sizeof(PAdicNumber));
    num->digits = (long*)calloc(length, sizeof(long));
    num->length = length;
    num->p = p;
    return num;
}

// Print p-adic number
void print_padic(PAdicNumber* num) {
    printf("(");
    for(int i = 0; i < num->length; i++) {
        printf("%ld", num->digits[i]);
        if(i < num->length - 1) printf(", ");
    }
    printf(")_%d", num->p);
}

// Convert a real number to p-adic representation
PAdicNumber* real_to_padic(double x, int p, int precision) {
    PAdicNumber* result = init_padic(p, precision);
    long integral = (long)floor(fabs(x) * pow(p, precision));
    
    for(int i = 0; i < precision && integral > 0; i++) {
        result->digits[i] = integral % p;
        integral /= p;
    }
    
    return result;
}

// P-adic norm calculation
double padic_norm(PAdicNumber* x) {
    int v = 0;
    while(v < x->length && x->digits[v] == 0) v++;
    return v == x->length ? 0.0 : pow(x->p, -v);
}

// Initialize p-adic signal structure
PAdicSignal* init_padic_signal(int num_samples, int p, int precision) {
    PAdicSignal* signal = (PAdicSignal*)malloc(sizeof(PAdicSignal));
    signal->samples = (PAdicNumber**)malloc(num_samples * sizeof(PAdicNumber*));
    signal->num_samples = num_samples;
    signal->p = p;
    
    for(int i = 0; i < num_samples; i++) {
        signal->samples[i] = init_padic(p, precision);
    }
    return signal;
}

// Convert real SDR samples to p-adic representation
PAdicSignal* convert_sdr_to_padic(double* samples, int num_samples, int p, int precision) {
    PAdicSignal* padic_signal = init_padic_signal(num_samples, p, precision);
    
    for(int i = 0; i < num_samples; i++) {
        padic_signal->samples[i] = real_to_padic(samples[i], p, precision);
    }
    return padic_signal;
}

// P-adic noise filtering
void padic_noise_filter(PAdicSignal* signal, double threshold) {
    printf("\nApplying p-adic noise filtering with threshold %f\n", threshold);
    int filtered = 0;
    
    for(int i = 0; i < signal->num_samples; i++) {
        double norm = padic_norm(signal->samples[i]);
        if(norm < threshold) {
            filtered++;
            for(int j = 0; j < signal->samples[i]->length; j++) {
                signal->samples[i]->digits[j] = 0;
            }
        }
    }
    
    printf("Filtered %d samples out of %d\n", filtered, signal->num_samples);
}

// Convert p-adic number back to real
double padic_to_real(PAdicNumber* num) {
    double result = 0;
    for(int i = 0; i < num->length; i++) {
        result += num->digits[i] * pow(num->p, i);
    }
    return result;
}

// Print signal samples
void print_signal_samples(PAdicSignal* signal, int start, int count) {
    printf("\nSignal samples (showing %d samples starting from %d):\n", count, start);
    printf("Sample Index | P-adic Representation | Real Value | P-adic Norm\n");
    printf("------------------------------------------------------------\n");
    
    for(int i = start; i < start + count && i < signal->num_samples; i++) {
        printf("%11d | ", i);
        print_padic(signal->samples[i]);
        printf(" | %10.4f | %10.4f\n", 
               padic_to_real(signal->samples[i]),
               padic_norm(signal->samples[i]));
    }
}

int main() {
    // Example parameters
    const int NUM_SAMPLES = 100;  // Reduced for clarity
    const int P = 7;
    const int PRECISION = 5;
    
    printf("Initializing SDR simulation with:\n");
    printf("Number of samples: %d\n", NUM_SAMPLES);
    printf("P-adic base (p): %d\n", P);
    printf("Precision: %d\n", PRECISION);
    
    // Generate sample SDR data
    double* sdr_samples = (double*)malloc(NUM_SAMPLES * sizeof(double));
    printf("\nGenerating sample SDR data (AM signal with noise)...\n");
    
    for(int i = 0; i < NUM_SAMPLES; i++) {
        // Simulate AM signal with noise
        sdr_samples[i] = sin(2 * M_PI * i / 50.0) * 
                        (1 + 0.5 * sin(2 * M_PI * i / 10.0)) + 
                        0.1 * ((double)rand() / RAND_MAX - 0.5);
    }
    
    // Print first few original samples
    printf("\nOriginal signal samples (first 5):\n");
    for(int i = 0; i < 5; i++) {
        printf("Sample %d: %f\n", i, sdr_samples[i]);
    }
    
    // Convert to p-adic representation
    printf("\nConverting to p-adic representation...\n");
    PAdicSignal* signal = convert_sdr_to_padic(sdr_samples, NUM_SAMPLES, P, PRECISION);
    
    // Print first few samples in p-adic form
    print_signal_samples(signal, 0, 5);
    
    // Apply p-adic noise filtering
    padic_noise_filter(signal, 0.1);
    
    // Print filtered samples
    printf("\nAfter noise filtering:\n");
    print_signal_samples(signal, 0, 5);
    
    // Calculate and print some statistics
    printf("\nSignal Statistics:\n");
    double max_norm = 0;
    double avg_norm = 0;
    for(int i = 0; i < signal->num_samples; i++) {
        double norm = padic_norm(signal->samples[i]);
        max_norm = fmax(max_norm, norm);
        avg_norm += norm;
    }
    avg_norm /= signal->num_samples;
    
    printf("Maximum p-adic norm: %f\n", max_norm);
    printf("Average p-adic norm: %f\n", avg_norm);
    
    // Clean up
    free(sdr_samples);
    for(int i = 0; i < NUM_SAMPLES; i++) {
        free(signal->samples[i]->digits);
        free(signal->samples[i]);
    }
    free(signal->samples);
    free(signal);
    
    return 0;
}
```
