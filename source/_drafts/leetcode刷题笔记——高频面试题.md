## 求众数：不同的数字两两抵消，剩下的就是众数。

int majorityElement(int* nums, int numsSize){
    int count = 1,major = nums[0];
    for (int i = 1; i<numsSize; i++)
    {
        if  (count == 0) 
        {
            major = nums[i];
            count ++;
        }
        else if (major == nums[i]) count++;
        else count--;
    }
    
    return major;
}

## 搜索二维矩阵：因为是有序矩阵，所以从右上或者左下开始

bool searchMatrix(int** matrix, int matrixRowSize, int matrixColSize, int target) {
    if (matrix == NULL || matrixRowSize < 1 || matrixColSize<1 ) return false;
    int i = 0;
    int j = matrixColSize-1;
    //最右上角开始查找，目标更大则说明在下一行，否则在左边；
    while(i<matrixRowSize && j>=0)
    {
        if (target == matrix[i][j]) return true;
        else if (target > matrix[i][j]) i++;
        else j--;
    }
    return false;
}

## 合并两个有序数组