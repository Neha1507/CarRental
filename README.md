# CarRental
@Entity
public class Car {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String brand;
    private String model;
    private boolean available;

    // Constructors, getters, setters
}

@Entity
public class Reservation {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private LocalDateTime startDate;
    private LocalDateTime endDate;

    @ManyToOne
    private Car car;

    // Constructors, getters, setters
}
//Repository
public interface CarRepository extends JpaRepository<Car, Long> {
    List<Car> findByAvailableTrue();
}

public interface ReservationRepository extends JpaRepository<Reservation, Long> {
    List<Reservation> findByCarAndEndDateAfter(Car car, LocalDateTime currentDate);
}
//Service
public class CarService {
    @Autowired
    private CarRepository carRepository;

    public List<Car> getAvailableCars() {
        return carRepository.findByAvailableTrue();
    }
}

@Service
public class ReservationService {
    @Autowired
    private ReservationRepository reservationRepository;

    public List<Reservation> getFutureReservationsForCar(Car car) {
        LocalDateTime currentDate = LocalDateTime.now();
        return reservationRepository.findByCarAndEndDateAfter(car, currentDate);
    }
}
//create controllers
@RestController
@RequestMapping("/cars")
public class CarController {
    @Autowired
    private CarService carService;

    @GetMapping("/available")
    public List<Car> getAvailableCars() {
        return carService.getAvailableCars();
    }
}

@RestController
@RequestMapping("/reservations")
public class ReservationController {
    @Autowired
    private ReservationService reservationService;

    @GetMapping("/{carId}/future")
    public List<Reservation> getFutureReservationsForCar(@PathVariable Long carId) {
        Car car = carService.getCarById(carId);
        return reservationService.getFutureReservationsForCar(car);
    }
}


