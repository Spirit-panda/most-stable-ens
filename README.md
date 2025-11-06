# most-stable-ens
deal with the stable structure
#!/bin/bash

output_file="ens5_results.txt"
structure_dir="structure"

# 创建structure目录
mkdir -p "$structure_dir"

echo "文件夹,最小值,ens编号" > "$output_file"

# 定义行号到ens编号的映射
declare -A ens_map
ens_map[2]="ens1"
ens_map[8]="ens2"
ens_map[14]="ens3"
ens_map[20]="ens4"
ens_map[26]="ens5"

for i in `seq 195 -5 175` `seq 174 -2 170` `seq 169 -1 160` `seq 158 -2 154` `seq 150 -5 140` `seq 138 -2 100` `seq 98 -2 10` `seq 9 -1 0`
do
    if [ -f "T$i/ens5" ]; then
        result=$(sed -n '2p;8p;14p;20p;26p' "T$i/ens5" | awk '{
            if(NR==1) print $2, 2
            else print $NF, (NR-1)*6+2
        }' | sort -n | head -1)
        
        min_value=$(echo "$result" | awk '{print $1}')
        line_num=$(echo "$result" | awk '{print $2}')
        ens=${ens_map[$line_num]}
        ens_num=${ens#ens}  # 去掉"ens"前缀，只保留数字
        
        echo "文件夹: $i, 最小值: $min_value, ens号: $ens"
        echo "T$i,$min_value,$ens" >> "$output_file"
        
        # 查找匹配的vesta文件
        vesta_file=$(find "T$i" -name "*en${ens_num}.vesta" -type f | head -1)
        
        if [ -n "$vesta_file" ] && [ -f "$vesta_file" ]; then
            # 复制文件到structure目录，保持原文件名
            cp "$vesta_file" "$structure_dir/"
            echo "  已复制: $vesta_file -> $structure_dir/$(basename "$vesta_file")"
        else
            echo "  未找到匹配的vesta文件: T$i/*en${ens_num}.vesta"
        fi
        
    else
        echo "文件夹: T$i, 没有找到 ens5 文件"
    fi
done

echo "结果已保存到 $output_file"
echo "结构文件已复制到 $structure_dir 目录"
