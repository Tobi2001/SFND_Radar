# SFND_Radak

## CFAR 2D

### Overview

The 2D CFAR process is implemented similarily to the 1D one described during the course:

```matlab
num_cells_T = (2 * (cells_Tr + cells_Gr) + 1) * (2 * (cells_Td + cells_Gd) + 1) ...
            - (2 * cells_Gr + 1) * (2 * cells_Gd + 1);
power_map = db2pow(RDM);
CFAR_output = zeros(size(RDM));
for i = 1 : size(RDM, 1) - (2 * (cells_Tr + cells_Gr))
    for j = 1 : size(RDM, 2) - (2 * (cells_Td + cells_Gd))
        noise_TGCUT = sum(power_map(i : i + 2 * (cells_Tr + cells_Gr), ...
                                    j : j + 2 * (cells_Td + cells_Gd)), ...
                          'all');
        noise_GCUT = sum(power_map(i + cells_Tr : i + cells_Tr + 2 * cells_Gr, ...
                                   j + cells_Td : j + cells_Td + 2 * cells_Gd), ...
                         'all');
        avg_noise = pow2db((noise_TGCUT - noise_GCUT) / num_cells_T);
        threshold = avg_noise + threshold_offset;
        cell_CUT = RDM(i + cells_Tr + cells_Gr, j + cells_Td + cells_Td);
        if cell_CUT > threshold
            CFAR_output(i + cells_Tr + cells_Gr, j + cells_Td + cells_Gd) = 1;
        end
    end
end  
```

The following input/output variables are used:

* _cells\_Tr_: Number of training cells in range dimension (one-sided)
* _cells\_Td_: Number of training cells in doppler dimension (one-sided)
* _cells\_Gr_: Number of guard cells in range dimension (one-sided)
* _cells\_Gd_: Number of guard cells in doppler dimension (one-sided)
* _num\_cells\_T_: The number of training cells
* _power\_map_: Converted values in _RDM_ from dB to power
* _CFAR\_output_: Final output matrix with filtered signal (signal values are 1, rest is 0)


### Description

The output map _CFAR\_output_ is initialized with the same dimensions as RDM and and all values are set to zero (this implicitely handles the edges after the thresholding).
Now a 2D-window is shifted over _power\_map_ and the threshold for the noise reduction is computed using the training cells around the currently processed cell (difference between area covering training-, guard- and CUT-cells and the one covering only guard- and CUT-cells; sum is divided by the total number of training cells to get the mean value). An offset is added to the threshold and then the CUT-cell is compared to the threshold. If the CUT-value is higher than the threshold the correlating cell in _CFAR\_output_ will be set to 1 (otherwise it will stay 0).

The numbers for the training cells, guard cells and the signal threshold offset were set to values working fine with the given properties in this project.

