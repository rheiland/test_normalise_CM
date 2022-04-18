#  test_normalise_CM

**Version:** 1.9.1

Note that we start with the `template` sample model, but then edit core/PhysiCell_standard_models.cpp per CM's slack msg:
```
if you add the following code block to standard_models.cpp just at the end of standard_update_cell_velocity function...
```

```
    // code from Cicely
    if (pCell->type_name == "cell")
    {
    std::cout << "At time " << PhysiCell_globals.current_time << " the cell has migration bias direction: " << pCell->phenotype.motility.migration_bias_direction << std::endl;
    double test = 0.0;
    test = pCell->phenotype.motility.migration_bias_direction[0]*pCell->phenotype.motility.migration_bias_direction[0];
    test += pCell->phenotype.motility.migration_bias_direction[1]*pCell->phenotype.motility.migration_bias_direction[1];
    test += pCell->phenotype.motility.migration_bias_direction[2]*pCell->phenotype.motility.migration_bias_direction[2];
    test = sqrt( test );
    std::cout << " -> the norm of this is " << test << std::endl;
    std::cout << "   ! BUT the nearest gradient vector is " <<  pCell->nearest_gradient(phenotype.motility.chemotaxis_index) << std::endl;
    double norm = 0.0;
    norm = pCell->nearest_gradient(phenotype.motility.chemotaxis_index)[0]*pCell->nearest_gradient(phenotype.motility.chemotaxis_index)[0];
    norm += pCell->nearest_gradient(phenotype.motility.chemotaxis_index)[1]*pCell->nearest_gradient(phenotype.motility.chemotaxis_index)[1];
    norm += pCell->nearest_gradient(phenotype.motility.chemotaxis_index)[2]*pCell->nearest_gradient(phenotype.motility.chemotaxis_index)[2];
    norm = sqrt( norm );
    pCell->nearest_gradient(phenotype.motility.chemotaxis_index) /= norm;
    std::cout << "     which has norm : " << norm << std::endl;
    if (norm > 1e-16){
        std::cout << "     this is above the normalisation threshold" << std::endl;
        std::cout << "     which means the normalised migration bias direction should be " << pCell->nearest_gradient(phenotype.motility.chemotaxis_index) <<  std::endl;
    }
    std::cout << std::endl;
    }
```
and I added to custom.cpp (create_cell_types()):
```
    cell_defaults.phenotype.motility.migration_bias_direction.assign({ 1,0, 0.0 });   //rwh
```

```
~/git/test_normalise_CM/PhysiCell_V.1.9.1$ make
~/git/test_normalise_CM/PhysiCell_V.1.9.1$ project rwh1.xml
...
At time 4 the cell has migration bias direction: 1 0 0 
 -> the norm of this is 1
   ! BUT the nearest gradient vector is -5.55112e-18 -1.11022e-17 0 
     which has norm : 1.24127e-17

~/git/test_normalise_CM/PhysiCell_V.1.9.1$ cd output
~/git/test_normalise_CM/PhysiCell_V.1.9.1/output$ cp ../beta/anim_svg.py .
~/git/test_normalise_CM/PhysiCell_V.1.9.1/output$ python anim_svg.py   # to plot cells moving migration bias direction
```
