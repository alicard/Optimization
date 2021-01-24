    // Knapsack
    public function optimization(){
        set_time_limit(4000);
        $start = microtime(true);

        $builder = $this->db->table('rating')
            ->select('idUser, COUNT(idBengkel) as c_idBengkel, AVG(nilai) as a_nilai')
            ->groupBy('idUser')
            ->limit(20);

        $query = $builder->get();
        $result = $query->getResult();

        // Store data to array
        foreach ($result as $key => $value) {
            // Store value from db to array
            $array_id[] = $value->idUser;
            $array_item[] = floatval($value->c_idBengkel);
            $array_rating[] = floatval($value->a_nilai);
        }

        /* -----------------------------------
            Get total possible combination
        ------------------------------------*/
        $total = count($array_item);
        $n = [$total];
        $faktorial_n = $n;

        for($i=0; $i<1; $i++) {
            for($j=1; $j<=$total; $j++){
                $r[$j] = $j; 

                // !n
                if($j<$total){
                    $faktorial_n[$i] *= $j;
                }
                
                // !r
                if($j<=$total){
                    $r[$j] = $j;
                }               
                
                // n-r
                $sub[$i] = $n[$i] - $j;
                if($sub[$i]==0){
                    $sub[$i] = 1;
                }
                $n_r[$j] = $sub[$i];
            }
        }

        # REVERSE ARRAY : $array = array_combine( array_keys( $array ), array_reverse( array_values( $array ) ) );
        # ADD FIRST ELEMENT : array_unshift($faktorial_n_r, 1);

        // !r
        for($i=1; $i<=$total; $i++){
            $jumlah_r[$i] = 1;
            for($j=1; $j<=$i; $j++){
                $jumlah_r[$i] *= $r[$j];
                $faktorial_r[$j] = $jumlah_r[$i];
            }
        }

        // !(n-r)
        for($i=1; $i<=$total; $i++){
            $jumlah_n_r[$i] = 1;
            for($j=$total; $j>=$i; $j--){
                $jumlah_n_r[$i] *= $n_r[$j];
                $faktorial_n_r[$i] = $jumlah_n_r[$i];
            }
        }

        // !n / !r !(n-r)
        for($a=0; $a<=0; $a++){
            for($j=1; $j<=$total; $j++){
                $combinations[$j] = @($faktorial_n[$a] / ($faktorial_r[$j] * $faktorial_n_r[$j]));
                //echo "!".$n[$a]." / !".$r[$j]." !(".$n[$a]." - ".$r[$j].") = ".$faktorial_n[$a]." / (".$faktorial_r[$j]." * ".$faktorial_n_r[$j].") = ".$combinations[$j]."</br>";
            }
        }

        // Get input here
        $W = $_POST['weight'];

        // Hasil total kemungkinan kombinasi
        $total_combinations = floatval(array_sum($combinations));
        if ($W==10) {
            $used_combinations = 1000;
        }
        elseif ($W==20) {
            $used_combinations = 5000;
        }
        else{
            $used_combinations = 10000;
        }

        /* ----------------------
            Start Brute Force 
        -----------------------*/
        // Mencari seluruh himpunan dari array database menggunakan binary count

        for ($i=0; $i<$used_combinations; $i++) {
            $sum_item[$i] = 0;
            $sum_rating[$i] = 0;
            $count[$i] = 0;

            // Check if each bit is set
            for ($j=0; $j<$used_combinations; $j++) {
                //Is bit $j set in $i?
                if(pow(2, $j) & $i){
                    // Get iduser and column
                    $id[$i][] = $array_id[$j];

                    // Sum array idUser per set 
                    $sum_item[$i] += $array_item[$j]; // Sum array idBengkel per set 
                    $sum_rating[$i] += $array_rating[$j]; // Sum array nilai per set 

                    // Store those into variable 
                    $item[$i] = $sum_item[$i]; 
                    $rating[$i] = $sum_rating[$i];

                    // Get size above array per column
                    $count_total_user = $array_item[$j];
                    $count_total_user = 1;
                    $total_user[$i] += $count_total_user;

                    // Divize rating by total user
                    $rating[$i] = $rating[$i]/$total_user[$i];
                }
            }   
            
            if($item[$i]==$W){
                // Find optimize 
                $find_optimization[] = array('id' => $id[$i], 'bobot' => $item[$i], 'nilai' => $rating[$i], 'total_user' => $total_user[$i]);
            }
        }
        
        $time_combine = microtime(true) - $start;
        echo "<br><br><br>Execution time for <i>Combination</i> and <i>Brute Force</i> in ".$time_combine." seconds<br>";
        $info = nl2br("Total Kombinasi = {$total_combinations}\nKombinasi digunakan = {$used_combinations}\nBobot = {$W}");
        $start_combine = microtime(true);

        //echo "<pre>". var_export($find_optimization, true). "</pre>";

        // Data optimize
        if(!empty($find_optimization))
        {
            sort($find_optimization);

            // Check max weight has multi element
            foreach ($find_optimization as $key => $value) {
                $rate[] = $value['nilai'];
            }

            foreach ($find_optimization as $key => $value) {
                if($value['nilai'] == max($rate)){
                    $optimization[] = $find_optimization[$key];
                    //$id_result[] = $value['id'];
                }
            }
        }

        if(!empty($optimization))
        {
            if(sizeof($optimization)>1)
            {
                $id = array_column($optimization, 'id');

                repeat:
                // Getting 2 array different value
                for($i=0; $i<count($optimization); $i++){
                    $array1 = array_values(array_diff($id[0], $id[1]));
                    $array2 = array_values(array_diff($id[1], $id[0]));
                }

                $mergeArray = array_merge($array1, $array2); 

                $userCheckRate = $this->db->table('rating')
                                    ->select('idUser, count(idBengkel) as c_idBengkel, AVG(nilai) as a_nilai')
                                    ->whereIn('idUser', $mergeArray)
                                    ->groupBy('idUser')
                                    ->get();

                $userResult = $userCheckRate->getResult();

                // Unset userid of array if has different values
                // Random userid of array if has same values
                if($userResult[0]->a_nilai > $userResult[1]->a_nilai){
                    //Array has no same values at all
                    unset($id[1]);
                }
                elseif ($userResult[0]->a_nilai < $userResult[1]->a_nilai) {
                    //Array has no same all values
                    unset($id[0]);
                }
                else{
                    if($userResult[0]->c_idBengkel > $userResult[1]->c_idBengkel){
                        //Array only has same nilai
                        unset($id[1]);
                    }
                    elseif ($userResult[0]->c_idBengkel < $userResult[1]->c_idBengkel) {
                        //Array only has same nilai
                        unset($id[0]);
                    }
                    else{
                        $randomKey = array_rand($mergeArray, 1);
                        //Array has same all values  
                        unset($id[$randomKey]);
                    }
                }

                // Reset indexing array
                $id = array_values($id);

                // Check if array > 1 then repeat block code
                if(sizeof($id)>1){
                    goto repeat;
                }
                else
                {
                    foreach ($id as $key => $value) {
                        $fixId = $value;
                    }

                    // Updating array to final result
                    foreach ($optimization as $key => $value) {
                        if($value['id'] == $fixId){
                            $finalOptimization[] = $optimization[$key];
                            $array_new_id = $value['id'];
                        } 
                    }
                    foreach ($array_new_id as $key => $value) {
                        $new_id[] = [
                                    'new_id' => $value
                                ];
                    } 
                }
            }
            else
            {
                foreach ($optimization as $key => $value) {
                    $finalOptimization[] = $optimization[$key];
                    $array_new_id = $value['id'];
                } 
                foreach ($array_new_id as $key => $value) {
                    $new_id[] = [
                                'new_id' => $value
                            ];
                } 
            }

            // Checking final result
            if(count($finalOptimization)>1){
                echo "Error occurred!";
                return false;
            }
            else
            {
                $newData = $this->db->table('optimization');
                            
                // Get DB table
                $getNewData = $this->db->table('optimization')
                                        ->select('*')->get();
                $result = $getNewData->getResult(); 

                // Checking table and replace old data
                if(empty($result)){
                    $newData->insertBatch($new_id);
                }
                else{
                    $newData->truncate($new_id);
                    $newData->insertBatch($new_id);
                }

                $time_multi = microtime(true) - $start_combine;
                echo "Execution time for <i>Multiple Data</i> has same value in ".$time_multi." seconds";

                return array('optimization' => $finalOptimization, 'info' => $info);
            }
        }
        // Return error no data
        else{
            $error = array(); // No data
            return array('optimization' => $error, 'info' => $info);
        }
    }
