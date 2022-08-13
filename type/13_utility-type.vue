<script setup lang="ts">
interface Product {
    id: number;
    name: string;
    price: number;
    brand: string;
    stock: number;
    something: any;
}


function fetchProducts(): Promise<Product[]> {
    
}

// 특정 삼품 상제 정보 노출 함수
// 원하는 값만 추출
type shoppingItem = Pick<Product, 'id' | 'name' | 'price'>
function displayProductDetail(shoppingItem: Pick<Product, 'id' | 'name' | 'price'> ) {
    
}
// 해당 타입을 제외한 나머지 추출
const productBook: Omit<Product, 'id' | 'price' | 'stock'> = {
    name: '티비',
    brand: '삼성',
    something: 5
}

type updateProcut = Partial<Product> // 옵셔널으로 생성
// 특정 상품 정보 업데이트 함수
function updateProcutItem(productItem: Partial<Product>) {

}

interface UserProfile {
    username: string;
    email: string;
    profilePhotoUrl: string;
}

// interface UserProfileUpdate {
//     username?: string;
//     email?: string;
//     profilePhotoUrl?: string;
// }

// type UserProfileUpdate = {   // 위와 같음 #1
//     username?: UserProfile['username'];
//     email?: UserProfile['email'];
//     profilePhotoUrl?: UserProfile['profilePhotoUrl'];
// }

// type UserProfileUpdate = {   // 위와 같음 #2
//     [p in 'username' | 'email' | 'profilePhotoUr']?: UserProfile[p]
// }
// type UserProfileKeys = keyof UserProfile

// type UserProfileUpdate = {   // 위와 같음 #3
//     [p in keyof UserProfile]?: UserProfile[p]
// }

// type Subset<T> = {              //위와 같음 #4
//     [p in keyof T]?: T[p]
// }
</script>