

#### 5. sm4 优化

sm4使用查表法与多线程、SIMD进行优化

该文件为「计算机系统原理」课程小组作业。
代码由小组共同完成，同组成员还有张济显、郭昊杉、冯信宇。

```c++

/*
	迭代32次
	参数：	u32 X[4]：迭代对象，保存结果    u32 RK[32]：轮密钥
	返回值：无
*/

void round_final(u32 x[], u32 RK[]) {
	short i;
	__m256i x0, x1, x2, x3, rk;
	__m256i t1;
	u32* a0,*a1,*a2,*a3;
	for (i = 0; i < 32; i=i+4) {
		x0 = _mm256_set_epi32(x[0], x[ 4], x[ 8], x[ 12], x[ 16], x[ 20], x[ 24], x[ 28]);
		x1 = _mm256_set_epi32(x[1], x[5], x[9], x[13], x[17], x[21], x[25], x[29]);
		x2 = _mm256_set_epi32(x[2], x[6], x[10], x[14], x[18], x[22], x[26], x[30]);
		x3 = _mm256_set_epi32(x[3], x[7], x[11], x[15], x[19], x[23], x[27], x[31]);
		rk = _mm256_set1_epi32(RK[i]);
		t1 = _mm256_xor_si256(x1, x2);
		t1 = _mm256_xor_si256(t1, x3);
		t1 = _mm256_xor_si256(t1, rk);
		t1 = T_simd(t1);
		x0 = _mm256_xor_si256(x0, t1);
		//x[(i + 4) % 4] = x[i % 4] ^ T(x[(i + 1) % 4] ^ x[(i + 2) % 4] ^ x[(i + 3) % 4] ^ RK[i], 1);
		
		rk = _mm256_set1_epi32(RK[i+1]);
		t1 = _mm256_xor_si256(x0, x2);
		t1 = _mm256_xor_si256(t1, x3);
		t1 = _mm256_xor_si256(t1, rk);
		t1 = T_simd(t1);
		x1 = _mm256_xor_si256(x1, t1);
	
		rk = _mm256_set1_epi32(RK[i+2]);
		t1 = _mm256_xor_si256(x0, x1);
		t1 = _mm256_xor_si256(t1, x3);
		t1 = _mm256_xor_si256(t1, rk);
		t1 = T_simd(t1);
		x2 = _mm256_xor_si256(x2, t1);
		
		rk = _mm256_set1_epi32(RK[i+3]);
		t1 = _mm256_xor_si256(x0, x1);
		t1 = _mm256_xor_si256(t1, x2);
		t1 = _mm256_xor_si256(t1, rk);
		t1 = T_simd(t1);
		x3 = _mm256_xor_si256(x3, t1);
		//x[(i + 4) % 4] = x[i % 4] ^ T(x[(i + 1) % 4] ^ x[(i + 2) % 4] ^ x[(i + 3) % 4] ^ RK[i], 1);
		a0 = (u32*)&x0;
		a1 = (u32*)&x1;
		a2 = (u32*)&x2;
		a3 = (u32*)&x3;
		for (int j = 0; j < 8; j++)
		{
			x[4 * j] = a0[j];
			x[1+4 * j] = a1[j];
			x[2+4 * j] = a2[j];
			x[3+4 * j] = a3[j];
		}
	}
}

void round_zhankai(u32 x[], u32 RK[]) {
	short i;
	for (i = 0; i < 32; i += 4) {
		//x[(i + 4) % 4] = x[i % 4] ^ T(x[(i + 1) % 4] ^ x[(i + 2) % 4] ^ x[(i + 3) % 4] ^ RK[i], 1);
		x[0] = x[0] ^ T(x[1] ^ x[2] ^ x[3] ^ RK[i], 1);
		x[1] = x[1] ^ T(x[2] ^ x[3] ^ x[0] ^ RK[i + 1], 1);
		x[2] = x[2] ^ T(x[3] ^ x[0] ^ x[1] ^ RK[i + 2], 1);
		x[3] = x[4] ^ T(x[0] ^ x[1] ^ x[2] ^ RK[i + 3], 1);
	}
}
/*
	反转函数
	参数；	u32 X[4]：反转对象    u32 Y[4]：反转结果
	返回值：无
*/

void reverse_simd(u32 x[], u32 y[]) {
	short i;
	for (i = 0; i < 4; i++) {
		y[i] = x[4 - 1 - i];
		y[i+4] = x[4 - 1 - i+4];
		y[i+8] = x[4 - 1 - i+8];
		y[i+12] = x[4 - 1 - i+12];
		y[i+16] = x[4 - 1 - i+16];
		y[i+20] = x[4 - 1 - i+20];
		y[i+24] = x[4 - 1 - i+24];
		y[i+28] = x[4 - 1 - i+28];
	}
}


void* sub_thread_final(void* args) {
	int xuhao = (int)args;
	int sub_num = NUM_P / NUM_THREADS;
	int begin = xuhao * sub_num;
	int end = (xuhao + 1) * sub_num;
	for (int i = begin; i < end; i=i+8)
	{
		round_final(X+4*i, RK);
		reverse_simd(X + 4 * i, Y + 4 * i);

	}

	return 0;

}
void encryptSM4_final() {
	pthread_t tids[NUM_THREADS];
	for (int i = 0; i < NUM_THREADS; ++i)
	{
		int ret = pthread_create(&tids[i], NULL, sub_thread_final, (void*)i);
	}
	//线程结束
	for (int i = 0; i < NUM_THREADS; i++)
	{
		pthread_join(tids[i], NULL);
	}
}




```
最终

